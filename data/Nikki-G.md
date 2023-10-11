### 1. Use cached length in loop.

In the function `_delegateMulti` loop condition always calls the function `Math.Max` to find the biggest number, but rather according to the design the that max value should ideally be equal to the `amountsLength`. So its better to use to the `amountsLength` instead of calculating maximum each time.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

#### Fix:
```
        for (
            uint transferIndex;
            transferIndex < amountsLength;
            transferIndex++
        ) { ... }
```


