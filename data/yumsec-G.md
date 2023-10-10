# Gas Optimization Report

## G-01: Compute Math.max inside a for-loop

### Lines of Code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

### Proof of Concept

Found in line 87 of ERC20MultiDelegate.sol

```solidity
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
) { ... }
```

### Mitigation

Cache the Math.max compute result outside the for-loop.

```solidity
uint256 maxOfSourcesAndTargetsLength = Math.max(sourcesLength, targetsLength);
for (
    uint transferIndex = 0;
    transferIndex < maxOfSourcesAndTargetsLength;
    transferIndex++
) { ... }
```

## G-02: Compute Math.min inside of the for-loop

### Lines of Code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98

### Proof of Concept

Found in line 98 of ERC20MultiDelegate.sol

```solidity
for (...) {
  // ...
  if (transferIndex < Math.min(sourcesLength, targetsLength)) { ... }
}
```

### Mitigation

Cache the Math.min compute result outside the for-loop.

```solidity
uint256 minOfSourcesAndTargetsLength = Math.min(sourcesLength, targetsLength);
for (...) {
  // ...
  if (transferIndex < minOfSourcesAndTargetsLength) { ... }
}
```