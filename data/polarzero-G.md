## [G-01] Use `amountsLength` in loop instead of unnecessary `Math.max(sourcesLength, targetsLength)`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80

### Description

In `ERC20MultiDelegate` > `_multiDelegate`, the `for` loop crawling through the given arrays to perform transfers uses `Math.max(sourcesLength, targetsLength)` as a reference for the max index.

However, this is unnecessary, since right above there is an equality check for `Math.max(sourcesLength, targetsLength) == amountsLength`. The latter would be cheaper to use.

### Impact

The current implementation wastes increasingly more gas as the given arrays get larger.

The following are some basic benchmarks of a single call to the `delegateMulti` function. The second column shows the gas usage with the current implementation, the third column shows the usage after replacing `Math.max(sourcesLength, targetsLength)` with `amountsLength`, and the fourth column calculates the difference.

| Array size | Current | Fixed | Difference |
| --- | --- | --- | --- |
| 1 target | 127,089 | 126,931 | 158 |
| 10 targets | 209,269 | 208,400 | 869 |
| 100 targets | 1,148,705 | 1,140,726 | 7,979 |

### Recommendation

Replace `Math.max(sourcesLength, targetsLength)` with `amountsLength` [on line 80](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80) of `ERC20MultiDelegate.sol`.