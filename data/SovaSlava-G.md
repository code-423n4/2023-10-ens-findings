## GAS report

[G1] Unnecessary 'if' in the last comparison
```diff
- } else if (transferIndex < targetsLength) {
+ } else {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L104-L106

[G2] The minimum value of two array's length can be cached
You can calculate the minimum length of arrays once and use this value, rather than calculating it at each iteration
```diff
  // Iterate until all source and target delegates have been processed.
+    uint minLen =  Math.min(sourcesLength, targetsLength);
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

-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex < minLen) {
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L84C2-L99C1

[G3]  If we understand that the array element is empty, then we do not need to assign the value address(0) to the variable because this is already the default value
```diff
-    address source = transferIndex < sourcesLength
-                ? address(uint160(sources[transferIndex]))
-                : address(0);
-    address target = transferIndex < targetsLength
-                ? address(uint160(targets[transferIndex]))
-                : address(0);
+    if(transferIndex < sourcesLength) {
+                source = address(uint160(sources[transferIndex]));
+    }
+    if(transferIndex < targetsLength) {
+                source = address(uint160(targets[transferIndex]));
+    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L92
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L95

[G4] Unnecessary conversion to bytes1
```diff
  bytes32 hash = keccak256(
            abi.encodePacked(
-               bytes1(0xff),
+               hex"ff",
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L208

[G5] Unnecessary conversion 0 to uint.
We can use msg.value (as 0) because function delegateMulti() is not payable. Value of msg.value always will be 0.
```diff
// case 1
-      if (bytecodeSize == 0) {
-            new ERC20ProxyDelegator{salt: 0}(token, delegate);
+      if (bytecodeSize == msg.value) {
+           new ERC20ProxyDelegator{salt: msg.value}(token, delegate);
// case 2
   bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
-               uint256(0), // salt
+               msg.value, // salt
                keccak256(bytecode)
            )
        );
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L210
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L185
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L186