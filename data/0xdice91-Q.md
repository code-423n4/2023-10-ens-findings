

| Number | Issue Detail | Severity |
|-----:|----|-----|
| [`L-01`](#L-01-merkle-leaf-value-should-not-be-64-bytes-before-hashing-in-merkleReward-distribution) | merkle leaf value should not be 64 bytes before hashing in MerkleReward Distribution | Low |
| [`L-02`](#L-02-use-safetransferFrom-instead-of-transferfrom) | Use safetransferFrom Instead of transferFrom. | Low |
| [`L-03`](#L-03-doesn't-implement-erc-1155-interface) | Doesn't implement ERC 1155 interface | Low |
| [`L-04`](#L-04-not-enough-coverage-of-the-codebase-in-the-docs) | Not enough coverage of the codebase in the docs | Low |


**Non Critical Issues List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| [`NC-01`](#NC-01-missing-events) | Missing Events | Non-Critical |
| [`NC-02`](#NC-02-no-check-if-delegators-are-approved) | No check if delegators are approved | Non-Critical |


# Low Risk Findings.

## L-01 Merkle leaf value should not be 64 bytes before hashing in MerkleReward Distribution
### Summary 
Merkle leaf value should not be 64 bytes before hashing
This library is out of Scope however something to note is that in [claimToken()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ENSToken.sol#L59C14-L59C25) the leaf is a hash of the msg.sender & amount however as stated in OZs library, the leaf should not be equal to 62 bytes.
```
    function claimTokens(uint256 amount, address delegate, bytes32[] calldata merkleProof) external {
        bytes32 leaf = keccak256(abi.encodePacked(msg.sender, amount));
        (bool valid, uint256 index) = MerkleProof.verify(merkleProof, merkleRoot, leaf);
```
you can see here that msg.sender and amount concatenated to the leaf, which is 64 bytes exactly
In Openzepplin library, we have the following comments
[link here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/02ea01765a9964541dd9cdcffc4a7f8b403c2ff6/contracts/utils/cryptography/MerkleProof.sol#L13)
```
 * WARNING: You should avoid using leaf values that are 64 bytes long prior to
 * hashing, or use a hash function other than keccak256 for hashing leaves.
 * This is because the concatenation of a sorted pair of internal nodes in
 * the merkle tree could be reinterpreted as a leaf value.
```
if an attacker controls two msg.sender address + the amount, he can claim the reward for free.


**This is meant to be of Medium severity but due to it being out of scope I think it's of a low severity finding however would love to see the judge's decisions, thanks and happy to help**
### Recommended Mitigation Step
Add one bytes to the hashed leaf 
## L-02 Use safetransferFrom instead of transferfrom
### Summary
In ERC20MultiDelegate.sol `transferFrom()` is used instead of `safeTransferFrom()`.Unless one is sure the given token reverts in case of a failure, it is necessary to check the `return values` of ERC20 token `transfers` or use something like `OpenZeppelin’s safeTransferFrom()` to ensure proper token transfers. This should have simply been reported in the bot findings, but somehow went undetected. It can be clearly seen in these functions;

[`_reimburse()`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L148)
```solidity
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

```
[`createProxyDelegatorAndTransfer()`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160)
```solidity
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
```
[`transferBetweenDelegators()`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L170)
```solidity
    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```
### Recommended Mitigation Step
Consider using OpenZeppelin `safeTransferFrom()` instead of `transferFrom()`.
## L-03 Doesn't implement ERC 1155 interface
### Summary
In OZ ERC 1155 it is recommended that any contract that makes use of the library should implement the receiver however this is not so in the codebase in scope, thereby not making the contract completely compatible.
### Recommended Mitigation Step
Implement OZ ERC 1155 receiver in the codebase in scope.
## L-04 Not enough coverage of the codebase in the docs
### Summary
On the contest page, there are no particular docs on the codebase in scope, and its functions supposed flows which could lead to an incorrect view when auditors or devs try to understand the protocol.
### Recommended Mitigation Step.
Create a docs that pinpoints the user/function flows of the codebase in scope


# Non Critical Issues.

## NC-01 Missing Events
### Summary
Certain functions do not haVe events do not have events for off-chain logs
- `_reimburse()`
- `transferBetweenDelegators()`
### Recommended Mitigation Step.
Emit events for the above functions.
## NC-02 No check if delegators are approved
### Summary
there is no check if the delegators are approved before tokens are transferred this breaks the second invariant of the protocol.
### Recommended Mitigation Step.
Check if delegators are approved