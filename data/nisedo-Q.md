# [L‑01] Missing checks for 0 address and 0 amount when setting input arrays

There are 3 instances:

[ERC20MultiDelegate.sol (#L57-L60):](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L60)
```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
```

# [L‑02] Two Transactions Required for Initial Delegation with `ERC20MultiDelegate` Contract
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160

To initiate a delegation for the first time via the `ERC20MultiDelegate` contract, a user must call the `delegateMulti()` function. This requires providing no source (or a 0 address), a single target to which voting power will be delegated, and the amount to delegate.

Internally, the `delegateMulti()` function invokes `_delegateMulti()`, which in turn calls `createProxyDelegatorAndTransfer()`. If no one has previously delegated to the specified target, a proxy is deployed, and the `token.transferFrom(msg.sender, proxyAddress, amount)` line is executed.

```solidity
        token.transferFrom(msg.sender, proxyAddress, amount);
```

This workflow necessitates that the user first approve the specified amount in a separate transaction sent to the ERC20 token contract before calling `delegateMulti()`. This could create a less-than-ideal user experience, especially if the ERC20 token used in conjunction with `ERC20MultiDelegate` does not support the `ERC20permit` extension.

## Impact
The need for two separate transactions can degrade the user experience, particularly for tokens that do not support the `ERC20permit` extension. This is especially relevant given that the sponsor indicated on Discord that the `ERC20MultiDelegate` contract could be used with any ERC20 token.

## Recommended Mitigation Steps
Ensure that the ERC20 token used in conjunction with `ERC20MultiDelegate` supports the `ERC20permit` extension. If it does not, users will be required to initiate two separate transactions to delegate their voting power.