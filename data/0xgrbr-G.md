# [G-01] State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas)
```
ERC20Votes public token;
```
```
ERC20Votes immutable public token;
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L28
```
Contract            ·  Method           ·  Min        ·  Max         ·  Avg   
ERC20MultiDelegate  ·  delegateMulti    ·      90194  ·      880257  ·     520486 // Before
ERC20MultiDelegate  ·  delegateMulti    ·      87650  ·      876625  ·     517319 // After
```

## 2. Cache variables
Cache Math.max and Math.min calculations before the loop.
The optimization is related to number of loops.

```
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..ce99006 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -70,6 +70,9 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256 sourcesLength = sources.length;
         uint256 targetsLength = targets.length;
         uint256 amountsLength = amounts.length;
+        uint256 mathMinSourceTargets = Math.min(sourcesLength, targetsLength); //@audit create cache variable
+        uint256 mathMaxSourceTargets = Math.max(sourcesLength, targetsLength); //@audit create cache variable
+
 
         require(
             sourcesLength > 0 || targetsLength > 0,
@@ -77,14 +80,14 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         );
 
         require(
-            Math.max(sourcesLength, targetsLength) == amountsLength,
+            mathMaxSourceTargets == amountsLength, //@audit use cache variable
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );
 
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < mathMaxSourceTargets; //@audit use cache variable
             transferIndex++
         ) {
             address source = transferIndex < sourcesLength
@@ -95,7 +98,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 : address(0);
             uint256 amount = amounts[transferIndex];
 
-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex < mathMinSourceTargets) { //@audit use cache variable
                 // Process the delegation transfer between the current source and target delegate pair.
                 _processDelegation(source, target, amount);
             } else if (transferIndex < sourcesLength) {
```
```
Contract            ·  Method           ·  Min        ·  Max         ·  Avg
ERC20MultiDelegate  ·  delegateMulti    ·      90194  ·      880257  ·     520486
ERC20MultiDelegate  ·  delegateMulti    ·      89893  ·      879642  ·     520068
```

## 3. Consider a _processDelegation reestructure.
The contract is calculating the targetProxyAddress twice. 
Since the function `transferBetweenDelegators` is internal and only called [here](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L134), we can reestructure a little bit the code to avoid the useless call.

```
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..df5ee9b 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -130,8 +130,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
 
         assert(amount <= balance);
 
-        deployProxyDelegatorIfNeeded(target);
-        transferBetweenDelegators(source, target, amount);
+        address targetProxyAddress = deployProxyDelegatorIfNeeded(target); //@audit reestructure
+        transferBetweenDelegators(source, targetProxyAddress, amount); //@audit reestructure call
 
         emit DelegationProcessed(source, target, amount);
     }
@@ -166,8 +166,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256 amount
     ) internal {
         address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-        address proxyAddressTo = retrieveProxyContractAddress(token, to);
-        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
+        // address proxyAddressTo = retrieveProxyContractAddress(token, to); //@audit supress one call
+        token.transferFrom(proxyAddressFrom, to, amount);
     }
```
```
Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·
ERC20MultiDelegate  ·  delegateMulti    ·      90194  ·      880257  ·     520486  ·
ERC20MultiDelegate  ·  delegateMulti    ·      90194  ·      880257  ·     517759  ·

```