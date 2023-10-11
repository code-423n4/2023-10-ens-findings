# QA Report

## Different ERC20MultiDelegate ids are mapped to the same address
The `_balances` of ERC1155 is a mapping from token id to owner => balance.
In [`function delegateMulti`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L61)
```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
```
we can think about `sources` and `targets` as desired token ids of type uint256, that will be converted to an address using `address(uint160(tokenId))` in
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L90-L95
```solidity
address source = transferIndex < sourcesLength
	? address(uint160(sources[transferIndex]))
	: address(0);
address target = transferIndex < targetsLength
	? address(uint160(targets[transferIndex]))
	: address(0);
```
This allows many valid token ids to be converted to the same address as long as the last 160 bits are  identical. This is due to the stripping of the leading bits during type casting.

When a token is minted, token ids are full uint256 numbers. The removed bits are not set to 0.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L113-L115
```solidity
if (targetsLength > 0) {
	_mintBatch(msg.sender, targets, amounts[:targetsLength], "");
}
```

So it's possible to create valid ERC20MultiDelegate tokens with different ids that will be mapped to the same `target` address. Which may be unexpected by a third parties and lead to accounting errors.

Please consider documenting this behavior.

## Delegating just received tokens in the same transaction will fail
Consider a user wants to transfer tokens from A to B and from B to C in one transaction. 
Because we burn before mint it's not possible, B does not yet have ERC20MultiDelegate tokens even so it has ERC20Votes tokens.

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L110-L115
```solidity
if (sourcesLength > 0) {
	_burnBatch(msg.sender, sources, amounts[:sourcesLength]);
}
if (targetsLength > 0) {
	_mintBatch(msg.sender, targets, amounts[:targetsLength], "");
}
```

It can be unexpected by a third party. Please consider documenting this behavior.

## The whole batch can be reverted by any `target` in it

[`_mintBatch`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L114) 
```solidity
_mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```
calls [`_doSafeBatchTransferAcceptanceCheck`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L229) 
```solidity
_doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, amounts, data);
```
which calls [`IERC1155Receiver(to).onERC1155BatchReceived`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L497)
```solidity
try IERC1155Receiver(to).onERC1155BatchReceived(operator, from, ids, amounts, data) returns (
```
Any `target` that will revert on this call (it can be programmed depending on blockchain state or it can change state in a front-run transaction) can censor the whole batch. It can be unexpected. 
Please consider documenting this behavior.

## External call may break internal accounting in 3rd parties
There is an external call to an untrusted contract from [`_mintBatch`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L114) 
```solidity
_mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```
calls [`_doSafeBatchTransferAcceptanceCheck`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L229) 
```solidity
_doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, amounts, data);
```
which calls [`IERC1155Receiver(to).onERC1155BatchReceived`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L497)
```solidity
try IERC1155Receiver(to).onERC1155BatchReceived(operator, from, ids, amounts, data) returns (
```

This can result in errors for third parties when they observe some ERC20Votes locked in the ERC20ProxyDelegator contracts without corresponding ERC20MultiDelegate tokens. Such discrepancies can give rise to incorrect assumptions regarding the active circulation and relative voting power of each ERC20Votes token. Additionally, this can lead to reentrancy issues in the delegateMulti function, potentially causing problems for third parties.

Please consider documenting this behavior.

## Constant should be used for salt
The salt is set to 0 in two lines:
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L186
```solidity
new ERC20ProxyDelegator{salt: 0}(token, delegate);
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L210
```solidity
uint256(0), // salt
```
Currently, the code uses "0" as a magic number, which can lead to errors if the salt value changes in the future. 
It is advisable to use a constant for this purpose.
