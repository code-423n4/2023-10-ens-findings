## Gas Optimization

| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | Use already cached value instead of calculating again  | 1 |
| [GAS-02](#GAS-02) | Cache value outside loop  | 1 |


## [GAS-01] Use already cached value instead of calculating again 

We already know the value of `Math.max(sourcesLength, targetsLength)` as it is calculated early on. But still the same check is carried inside the loop to check the length till which `transferIndex` will be executed.

```solidity
File: ERC20MultiDelegate.sol

80:      Math.max(sourcesLength, targetsLength) == amountsLength,    /// @note - we already know the value of the max function


- 87:    transferIndex < Math.max(sourcesLength, targetsLength);
+ 87:    transferIndex < amountsLength
```

## [GAS-02] Cache value outside loop

Variables should be cached outside loops to save gas & avoid unnecessary computations.

```solidity
File: ERC20MultiDelegate.sol

+ 83:    minLength = Math.min(sourcesLength, targetsLength);
         ..........

- 98:  if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+ 98:  if (transferIndex < minLength) {
```
