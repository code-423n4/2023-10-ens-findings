# GAS Optimization Report


## G-001 - Caching Value of calculation with unchanging values could save gas

The `delegateMulti` function of `ERC20MultiDelegate` is calculating the maximum of `sourcesLengtt` and `targetsLength` multiple times.
First time it is calculated inside the require check to see if at least one of the arrays is not empty ([Line 80](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L80)).

Later is caclulated again in each iteration of the loop as the loop exit condition ([Line 87](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87)).

It would be more gas efficient if the max value is calculated once and stored in a memory variable to reuse it later.

The same goes for the calculation of the min value of these to lengths. It is calculated in every iteration of the for loop ([Line 98](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98))


### Tools used

- manual inspection of the code
- Hardhat

## Recommendation 

Before the loop and the require statment calculate the values once like this:

```solidity
uint256 sourceTargetMax = Math.max(sourcesLength, targetsLength);
uint256 sourceTargetMin = Math.min(sourcesLength, targetsLength);
```
Then change the lines like this:
line 80:    `sourceTargetMax == amountsLength,`
line 87:    `transferIndex < sourceTargetMax;`
line:98:    `if (transferIndex < sourceTargetMin) {`

