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

## G-03: transferIndex++ should be unchecked{transferIndex++} when it is impossible for them to overflow

### Lines of Code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L88

### Proof of Concept

Found in line 88 of ERC20MultiDelegate.sol.
Since `Math.max(sourcesLength, targetsLength) == amountsLength`, which is uint256, it's impossible for `transferIndex` to overflow. We could use `unchecked{transferIndex++}` to save gas.

```solidity
for (
      uint transferIndex = 0;
      transferIndex < Math.max(sourcesLength, targetsLength);
      transferIndex++
    ) {
    ...
    }
```

### Mitigation

Use `unchecked{transferIndex++}` instead of `transferIndex++`.

```solidity
for (
      uint transferIndex = 0;
      transferIndex < Math.max(sourcesLength, targetsLength);
      unchecked{transferIndex++}
    ) {
    ...
    }
```