## [G-01] Using `amountsLength` to replace `Math.max(sourcesLength, targetsLength)` in loop to save gas

Since the `require(Math.max(sourcesLength, targetsLength) == amountsLength)` can ensure that `amountsLength == Math.max(sourcesLength, targetsLength)`. Convert `transferIndex < Math.max(sourcesLength, targetsLength)` to `transferIndex < amountsLength` can save gas.

### Proof Concept
```
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
)
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L86-L88