# Calculating Math functions for every iteration

While looping through the arrays, the Math.max and Math.min is checked on each iteration, this would cost less if it was checked only once before looping starts as the length of the arrays do not change. 

Locations where they exist:

* Line 87

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

```
       for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )
```
* Line 98

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98

```
if (transferIndex < Math.min(sourcesLength, targetsLength))
```

## Risk
No identified risk, however due to each check on each iteration, the gas cost would be increased.

## Mitigation

Check and declare the Math.max and Math.min in a variable before starting iteration.




