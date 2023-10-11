## [L-01] Missing checks for `addrees(0)` in `targets` of function `_delegateMulti`
The function `_delegateMulti` does not check `address(0)` in `targets`. If the user call the function with `targets = [address(0)]`, the ENSToken would not distribute the corrosponding votes to `address(0)`;

### Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L82

## [N-01] `assert(amount <= balance)` is not necessary

The function `_processDelegation` checks the ERC1155 token balance of `source`, but this check is not necessary. This is because the `_burnBath(msg.sender, sources, amounts[:sourcesLength])` in function `_delegateMulti` would make the transaction revert if `amount <= balance`.

### Proof Concept
```
    uint256 balance = getBalanceForDelegate(source);

    assert(amount <= balance);
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L129-L131