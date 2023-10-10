# [G-01] Call balanceOf internally 
[ERC20MultiDelegate.sol#L195](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L195) 

```diff
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
-       return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+       return balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```
# [G-02] function _delegateMulti - avoid if-else and ternary operator abuse by splitting the loop into smaller ones
[ERC20MultiDelegate.sol#L85-L108](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L108)
Instead of a single "swiss-army-knife" loop that handles every input via if-else and ternary operators, it will be better to separate the loop into smaller ones.

The expected input is: a amounts, s sources, t targets; a = max(s,t). 
```
        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
For `transferIndex` ∈ `[0; min(sourcesLength, targetsLength))`, only function `_processDelegation` is called. So we're moving it into a separate loop, avoiding plenty of branchings:
```
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }
```
For `transferIndex` ∈ `[min(sourcesLength, targetsLength), max(sourcesLength, targetsLength))`, we create a separate loop (one for sources and one for targets):
```
        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```

Original version:

```
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
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
        }
```
Optimized version:
```
        uint256 minSourcesTargetsLength = Math.min(
            sourcesLength,
            targetsLength
        );
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }

        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```