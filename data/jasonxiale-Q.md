# [N-01] `ERC20MultiDelegate._processDelegation` can add `source != target` check
[ERC20MultiDelegate._processDelegation](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124-L137) should add the following check to return early if **source == target**
```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..c2360b1 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -126,6 +126,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address target,
         uint256 amount
     ) internal {
+        if(source == target ) return;
         uint256 balance = getBalanceForDelegate(source);
 
         assert(amount <= balance);
```

# [N-02] `ERC20MultiDelegate.delegateMulti` can rewrite to save more gas
[ERC20MultiDelegate.delegateMulti](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65-L116) can be re-write as following to make the code more readable, and save some gas 
```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..8c6c2fc 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -71,38 +71,40 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256 targetsLength = targets.length;
         uint256 amountsLength = amounts.length;

+        uint maxLength = Math.max(sourcesLength, targetsLength);
+        uint minLength = Math.min(sourcesLength, targetsLength);
+
         require(
             sourcesLength > 0 || targetsLength > 0,
             "Delegate: You should provide at least one source or one target delegate"
         );

         require(
-            Math.max(sourcesLength, targetsLength) == amountsLength,
+            maxLength == amountsLength,
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );

         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < maxLength;
             transferIndex++
         ) {
-            address source = transferIndex < sourcesLength
-                ? address(uint160(sources[transferIndex]))
-                : address(0);
-            address target = transferIndex < targetsLength
-                ? address(uint160(targets[transferIndex]))
-                : address(0);
             uint256 amount = amounts[transferIndex];
+            if(amount == 0) continue;

-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex < minLength) {
                 // Process the delegation transfer between the current source and target delegate pair.
+                address source = address(uint160(sources[transferIndex]));
+                address target = address(uint160(targets[transferIndex]));
                 _processDelegation(source, target, amount);
             } else if (transferIndex < sourcesLength) {
                 // Handle any remaining source amounts after the transfer process.
+                address source = address(uint160(sources[transferIndex]));
                 _reimburse(source, amount);
             } else if (transferIndex < targetsLength) {
                 // Handle any remaining target amounts after the transfer process.
+                address target = address(uint160(targets[transferIndex]));
                 createProxyDelegatorAndTransfer(target, amount);
             }
         }
```
