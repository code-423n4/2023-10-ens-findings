## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-current-design-of-delegatemulti-might-limit-its-functionality) | Current design of `delegateMulti()` might limit its functionality | Low |
| [L-02](#l-02-smart-contract-accounts-might-not-be-compatible-with-erc20multidelegate) | Smart contract accounts might not be compatible with `ERC20MultiDelegate` | Low |
| [N-01](#n-01-unnecessary-external-call-in-getbalancefordelegate) | Unnecessary external call in `getBalanceForDelegate()` | Non-Critical |


## [L-01] Current design of `delegateMulti()` might limit its functionality

The current implementation of `delegateMulti()` performs three kinds of vote transfers:
- If `sources[i]` and `targets[i]` are both non-zero, it transfers votes between the source and target delegatee.
- If only `sources[i]` is non-zero, it transfers vote tokens from the delegatee to the caller.
- If only `targets[i]` is non-zero, it transfers vote tokens from the caller to the delegatee.

This can be seen in its implementation:

[ERC20MultiDelegate.sol#L98-L107](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98-L107)

```solidity
            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
```

However, such a design only allows caller - delegatee transfers after all delegatee - delegatee transfers have been performed. 

This limits the functionality of the contract as vote delegations cannot be performed in certain orders. For example:

1. Transfer 100 votes from the caller to delegatee A. (caller - delegatee)
2. Transfer 50 votes from delegatee A to B. (delegatee - delegatee)

If a user wishes to perform the vote transfers above, he will be unable to do so due to the current design.

### Recommendation

Consider refactoring the function's implementation, such that users can perform caller - delegatee transfers 

## [L-02] Smart contract accounts might not be compatible with `ERC20MultiDelegate`

The `delegateMulti()` function calls `_mintBatch()` to mint ERC-1155 tokens to the caller:

[ERC20MultiDelegate.sol#L113-L115](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L113-L115)

```solidity
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
```

`_mintBatch()` performs an external callback, which calls `onERC1155Received()` on the `to` address if it is a contract:

[ERC1155.sol#L320](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L320)

```solidity
        _doSafeBatchTransferAcceptanceCheck(operator, address(0), to, ids, amounts, data);
```

This might cause `delegateMulti()` to always revert for existing contracts that do not implement `onERC1155Received()`, such as old smart contract wallets.

### Recommendation

Consider overriding `_mintBatch()` to remove the `_doSafeBatchTransferAcceptanceCheck()` call.

## [N-01] Unnecessary external call in `getBalanceForDelegate()`

`getBalanceForDelegate()` performs an external call to the contract's `balanceOf()` function:

[ERC20MultiDelegate.sol#L192-L196](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L192-L196)

```solidity
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```

However, this is unnecessary as `balanceOf()` is declared as a `public` function:

[ERC1155.sol#L70](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC1155/ERC1155.sol#L70)

```solidity
    function balanceOf(address account, uint256 id) public view virtual override returns (uint256) {
```

### Recommendation

Consider calling `balanceOf()` directly:

[ERC20MultiDelegate.sol#L192-L196](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L192-L196)

```diff
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
-       return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+       return balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```