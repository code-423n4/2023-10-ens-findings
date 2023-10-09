# Calling function on each iteration is expensive

This [for loop](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L109) calls **Math.max** after each iteration. 

```jsx
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
Consider caching the result in a variable and using that variable for the condition: 

```jsx
        uint256 max = Math.max(sourcesLength, targetsLength);
        for (
            uint transferIndex = 0;
            transferIndex < max;
            transferIndex++
        ) {
```