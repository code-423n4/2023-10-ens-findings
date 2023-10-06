### Report 1:
Continuously recalling "Math.min(sourcesLength, targetsLength)" will use too much unnecessary gas, it should be cached outside the loop first before usage.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98
```solidity
    function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
... 
+++ uint256 minLength = Math.min(sourcesLength, targetsLength)
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
...
--- if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+++ if (transferIndex < minLength ) {
...
```