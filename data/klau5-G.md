# Duplicated function call at for loop

## Links to affected code

[https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87)

## Proof of Concept

`Math.max(sourcesLength, targetsLength)` and `Math.min(sourcesLength, targetsLength)` results are always same at loop, so it would be better to use local variable.

```solidity
for (
    uint transferIndex = 0;
@>  transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
) {
    address source = transferIndex < sourcesLength
        ? address(uint160(sources[transferIndex]))
        : address(0);
    address target = transferIndex < targetsLength
        ? address(uint160(targets[transferIndex]))
        : address(0);
    uint256 amount = amounts[transferIndex];

@>  if (transferIndex < Math.min(sourcesLength, targetsLength)) {
        // Process the delegation transfer between the current source and target delegate pair.
        _processDelegation(source, target, amount);
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        _reimburse(source, amount);
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        createProxyDelegatorAndTransfer(target, amount);
    }
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
+ uint256 maxLength = Math.max(sourcesLength, targetsLength);
+ uint256 minLength = Math.min(sourcesLength, targetsLength);

for (
    uint transferIndex = 0;
-   transferIndex < Math.max(sourcesLength, targetsLength);
+   transferIndex < maxLength;
    transferIndex++
) {
    ...

-   if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+   if (transferIndex < minLength ) {
        // Process the delegation transfer between the current source and target delegate pair.
        _processDelegation(source, target, amount);
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        _reimburse(source, amount);
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        createProxyDelegatorAndTransfer(target, amount);
    }
}
```