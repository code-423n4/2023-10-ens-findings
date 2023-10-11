# [L-01] Users can mint different ERC-1155 tokenIds representing the same delegate
```
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
```
    _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```
Because _delegateMulti casts sources and targets from uint256 -> uint160 -> address, but _mintBatch uses the original uint256s, it is possible to mint up to 2^96 different tokens which would represent the same delegate - all uint256s with the same `mod 2^160 == uint160(delegate)`.

These tokens are possible to transfer to another users or ERC1155TokenReceivers, as any other ERC1155 tokens. 

But it in order to use them for transferring voting power between delegates, a user will need to have at least the same amount of the "normal" token (with `id == uint256(uint160(delegate))`), because `assert` checks for the truncated id balance:

```
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);
        assert(amount <= balance);
``` 
## Impact
Such users that entered wrong address as a target, unintentionally or maliciously, may only harm themselves, and only in the way that they may not be able to transfer voting power between delegates. The affected users will just have to call delegateMulti with `source = tokenId` and empty destination, which will successfully withdraw their tokens.

It is also inconvenient for anyone to query delegators' balances: calling `ERC1155#balanceOf(delegator, (uint256(uint160(delegate)))` may not show the complete picture, as there could be many tokens representing the same (ERC20Votes,delegate) pair.

## Recommended Mitigation

Use SafeCast for downcasting uint256 -> uint160 to prevent minting tokens with id >= 2^160.

# [N-01] library `Address` is imported but never used
[ERC20MultiDelegate.sol#L4](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4)
```
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
```
[ERC20MultiDelegate.sol#L26](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L26)
```
    using Address for address;
```

# [N-02] Consider emitting DelegationProcessed event in `_reimburse` and `createProxyDelegatorAndTransfer`
```
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
Event `DelegationProcessed` is emitted whenever tokens are moved from one proxy to another, but not emitted when users reimburse their tokens or delegate their own tokens to a new delegate.

## Recommendation
```diff
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
+       emit DelegationProcessed(source, msg.sender, amount);
    }
```
```
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
+       emit DelegationProcessed(msg.sender, target, amount);
    }
```
`address(0)` instead of `msg.sender` could also be used, but `msg.sender` would be more informative and convenient as it would show the delegator.