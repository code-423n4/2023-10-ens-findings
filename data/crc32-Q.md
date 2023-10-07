### Cache result of function call in stack variable outside of `for`-loop
```solidity
if (transferIndex < Math.min(sourcesLength, targetsLength)) {
```
`Math.min(...)` every time returns the same result in this case, so it could be cached outside of loop in a stack variable.
For example the minimum between 1 and 2 is 1, so no need to recalculate the minimum multiple times inside the loop.
Relevant code line:
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98

