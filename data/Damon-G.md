## [G-01] Using `amountsLength` to replace `Math.max(sourcesLength, targetsLength)` in loop to save gas

Since the `require(Math.max(sourcesLength, targetsLength) == amountsLength)` can ensure that `amountsLength == Math.max(sourcesLength, targetsLength)`. Convert `transferIndex < Math.max(sourcesLength, targetsLength)` to `transferIndex < amountsLength` can save gas.

### Proof Concept
```
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
)
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L86-L88

## [G-02] Setting the `constructor` to `payable` to save gas

### Proof Concept
```
constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) {
        token = _token;
}
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L44-L49

## [G-03] In function `_delegateMulti`, Loading `source` and `target` in `if` statement to save gas

Since `if (transferIndex < sourcesLength)` has the same judgement as the above `address source = transferIndex < sourcesLength ? address(uint160(sources[transferIndex])): address(0);`, move the loading of `source` into the `if (transferIndex < sourcesLength)` could save gas.

Here is the optimizated code
```
    uint256 amount = amounts[transferIndex];

    if (transferIndex < Math.min(sourcesLength, targetsLength)) {
        // Process the delegation transfer between the current source and target delegate pair.
        address source = address(uint160(sources[transferIndex]));
        address target = address(uint160(targets[transferIndex]));
        _processDelegation(source, target, amount);
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        address source = address(uint160(sources[transferIndex]));
        _reimburse(source, amount);
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        address target = address(uint160(targets[transferIndex]));
        createProxyDelegatorAndTransfer(target, amount);
    }
```

### Proof Concept
```
    address source = transferIndex < sourcesLength
        ? address(uint160(sources[transferIndex]))
        : address(0);
    address target = transferIndex < targetsLength
        ? address(uint160(targets[transferIndex]))
        : address(0);
    uint256 amount = amounts[transferIndex];

    if (transferIndex < Math.min(sourcesLength, targetsLength)) {
        // Process the delegation transfer between the current source and target delegate pair.
        _processDelegation(source, target, amount);
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        _reimburse(source, amount);
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        createProxyDelegatorAndTransfer(target, amount);
    }
```