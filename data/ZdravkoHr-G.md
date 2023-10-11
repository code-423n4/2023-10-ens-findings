# Calling function on each iteration is expensive

This [for loop](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L109) calls **Math.max** and **Math.min** on each iteration. 

```jsx
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
                // @audit-issue - can be called by anyone?
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```
Consider caching these before the start of the loop:

```jsx
        uint256 max = Math.max(sourcesLength, targetsLength);
        uint256 min = Math.min(sourcesLength, targetsLength);
        for (
            uint transferIndex = 0;
            transferIndex < max;
            transferIndex++
        ) {
           ...
           if (transferIndex < min) {
           ...
        }

```