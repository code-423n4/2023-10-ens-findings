## GAS report of ENS project.

### [G-1] Unnecessary 'if' in the last comparison.
In the last comparison there is no need to write 'else if', because if we have reached this point, it means that transferIndex is definitely less than targetsLength. And you can simply write 'else'.
```diff
     if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
-            } else if (transferIndex < targetsLength) {
+            } else {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
             }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L104-L106

### [G-2] The minimum value of two array's length can be cached.
You can calculate the minimum length of arrays once and use this value, rather than calculating it at each iteration.
```diff
  // Iterate until all source and target delegates have been processed.
+    uint minLen =  sourcesLength < targetsLength ? sourcesLength: targetsLength;
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

### [G-3]  Unnecessary assignment of zero value.
If we understand that the array element is empty, then we do not need to assign the value address(0) to the variable because this is already the default value.
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

### [G-4] Unnecessary conversion to bytes1.
It is not necessary to convert the hex value into bytes1. You can specify it through hex"ff" (without 0x prefix).
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

### [G-5] Dont use MATH library for simple math.
It is not necessary to use library functions for simple mathematics. We can insert the calculation directly into the contract. And create variable maxLen, and use it instead call library function.
```diff
// Create variable maxLen
+  uint maxLen = (sourcesLength >= targetsLength ? sourcesLength : targetsLength);
// place 1

require(
-     Math.max(sourcesLength, targetsLength) == amountsLength,
+     maxLen == amountsLength,
      "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
);

// place 2
 for (
            uint transferIndex = 0;
-           transferIndex < Math.max(sourcesLength, targetsLength);
+           transferIndex < maxLen;
            transferIndex++
    ) {

```

### [G-6] Do not import unused library.
The contract does not use functions from the Address library, so we can remove the line with applying library to type address and remove import the library to contract. This will reduce the cost of deploying a contract.
```diff
- import {Address} from "@openzeppelin/contracts/utils/Address.sol";
  import "@openzeppelin/contracts/access/Ownable.sol";
  import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
  import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
  import "@openzeppelin/contracts/utils/math/Math.sol";

contract ERC20MultiDelegate is ERC1155, Ownable {
-    using Address for address;
+     // delete line

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L26

### [G-7] Unnecessary balance check.
There is no need to check additionally the balance in the _processDelegation() function, because this point is checked in the token burning function - _burnBatch(). If the user specifies a larger balance than he delegated, the contract will not be able to burn more tokens than he has (ERC-1155).

We can delete assert() and function getBalanceForDelegate(). So, it will decrease deploy cost.
```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
-        uint256 balance = getBalanceForDelegate(source);

-        assert(amount <= balance);


-    function getBalanceForDelegate(
-        address delegate
-    ) internal view returns (uint256) {
-        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
-    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131

### [G-8] All functions not payable, so we can change in all places 0 to msg.value.
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

