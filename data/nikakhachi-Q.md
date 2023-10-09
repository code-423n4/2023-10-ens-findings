# [01] `assert` without the error message

It's better to include some kind of error message when the revert occurs, to quickly analyze the underlying reason.

```
File: ERC20MultiDelegate.sol

129:        uint256 balance = getBalanceForDelegate(source);
130:
131:        assert(amount <= balance);
```
Can be changed to:
```
assert(amount <= balance, "Insufficient Delegate Balance");       
```