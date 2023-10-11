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

## [N-02] Only the delegator could handle the voting power of their delegates.

The `_burnBath(msg.sender, ...)` ensures that only the delegator could handle the voting power of their delegates. However, in the test case `should be able to delegate to already delegated delegates` of test file `delegatemulti.js`, it try to use `secondDelegator` (not developer) to invoke function `delegateMulti` for delegating. This test case should fail to pass since only developer could handle their voting power.

Actually, the `amountsForSecondary` is zero, the function `delegateMulti` does not transfer any tokens. So the execution could finish without revert.

The test case is as follows.
```
    const [_, secondDelegator] = await ethers.getSigners();
    const secondDelegatorBalance = await token.balanceOf(
    secondDelegator.address
    );

    await token
    .connect(secondDelegator)
    .approve(multiDelegate.address, secondDelegatorBalance);

    const amountsForSecondary = delegates.map(() =>
    secondDelegatorBalance.div(delegates.length)
    );

    await multiDelegate
    .connect(secondDelegator)
    .delegateMulti([], delegates, amountsForSecondary);
```

### Proof of Concept
```
    if (sourcesLength > 0) {
        _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
    }
    if (targetsLength > 0) {
        _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
    }
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L110-115