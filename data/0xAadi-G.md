# GAS Report

## Summary

### Gas Optimizations

Total **2 instances** over **2 issues**with **432** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G&#x2011;01]](#g01-amountsLength-can-be-used-istead-of-max-calculation) | `amountsLength` can be used instead of `Math.max(sourcesLength, targetsLength)` in the for loop | 1 | 308 |
| [[G&#x2011;02]](#g02-min-calculation-can-be-cached-to-save-gas) | `Math.min(sourcesLength, targetsLength)` can be cached to save gas | 1 | 124 |

## Gas Optimizations

### [G&#x2011;01] `amountsLength` can be used instead of `Math.max(sourcesLength, targetsLength)` in the for loop

The require statement at lines [79-82](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L74C1-L77C11) ensures that `amountsLength` is equal to `Math.max(sourcesLength, targetsLength)`, you can replace `Math.max(sourcesLength, targetsLength)` with `amountsLength` in the for loop. This eliminates the need for the max calculation in every iteration, resulting in gas savings.

There are 1 instances:

- *ERC20MultiDelegate.sol* ( [#L84](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87)):

```solidity
84:        // Iterate until all source and target delegates have been processed.
85:        for (
86:            uint transferIndex = 0;
87:            transferIndex < Math.max(sourcesLength, targetsLength);
88:            transferIndex++
89:        ) {
```

#### Gas Savings for ```ERC20MultiDelegate.delegateMulti```, obtained via protocol’s tests: Avg 308 gas

```
·······················|···················|·············|··············|·············|···············|··············
|                      ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|       Before         ·  delegateMulti    ·      90194  ·      880257  ·     520486  ·           19  ·       4.97  │
·······················|···················|·············|··············|·············|···············|··············
|       After          ·  delegateMulti    ·      89939  ·      879832  ·     520178  ·           19  ·       4.96  │
·······················|···················|·············|··············|·············|···············|··············
```

```diff
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength); // @audit `amountsLength` can be used instead of `Math.max(sourcesLength, targetsLength)`
+            transferIndex < amountsLength; 
             transferIndex++
         ) {

```

### [G&#x2011;02] `Math.min(sourcesLength, targetsLength)` can be cached to save gas

`sourcesLength` and `targetsLength` remain unchanged within the loop, the value of `Math.min(sourcesLength, targetsLength)` is a constant throughout the loop iterations. Therefore, there is no need to calculate the minimum value in each iteration. So the `Math.min(sourcesLength, targetsLength)` can be cached before the loop and it can be used instead of the min calculation in every iteration, which will saves gas costs.

There are 1 instances:

- *ERC20MultiDelegate.sol* ( [#L98](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98C12-L98C74)):

```solidity
98:            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
```

#### Gas Savings for ```ERC20MultiDelegate.delegateMulti```, obtained via protocol’s tests: Avg 124 gas

```
·······················|···················|·············|··············|·············|···············|··············
|                      ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|       Before         ·  delegateMulti    ·      90194  ·      880257  ·     520486  ·           19  ·       4.97  │
·······················|···················|·············|··············|·············|···············|··············
|       After          ·  delegateMulti    ·      90135  ·      880054  ·     520362  ·           19  ·       4.97  │
·······················|···················|·············|··············|·············|···············|··············
```

```diff
@@ -81,6 +81,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );
 
+        uint256 minLength = Math.min(sourcesLength, targetsLength);
+
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
@@ -95,7 +97,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 : address(0);
             uint256 amount = amounts[transferIndex];
 
-            if (transferIndex < Math.min(sourcesLength, targetsLength)) { @audit `Math.min(sourcesLength, targetsLength)` can be cached
+            if (transferIndex < minLength) {
                 // Process the delegation transfer between the current source and target delegate pair.
                 _processDelegation(source, target, amount);
             } else if (transferIndex < sourcesLength) {

```