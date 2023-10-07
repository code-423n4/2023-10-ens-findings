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
# [G-02] Cache values that are used multiple times
[ERC20MultiDelegate.sol#L79-L108](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L79-L108)
```diff
+       uint256 maxSourcesOrTargetsLength = Math.max(sourcesLength, targetsLength);
        require(
-           Math.max(sourcesLength, targetsLength) == amountsLength,
+           maxSourcesOrTargetsLength == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

+       uint256 maxSourcesOrTargetsLength = Math.max(sourcesLength, targetsLength);
+       uint256 minSourcesOrTargetsLength = Math.min(sourcesLength, targetsLength);
        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-           transferIndex < Math.max(sourcesLength, targetsLength);
+           transferIndex < maxSourcesOrTargetsLength;
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

-           if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+           if (transferIndex < minSourcesOrTargetsLength) {
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