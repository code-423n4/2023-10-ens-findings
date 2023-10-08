## [N-01] Unnecessary import of library

The Openzeppelin Address library is never used.

### Recommended Mitigation

Consider removing the Address library import.

```diff
diff --git a/contracts/ERC20MultiDelegateOrig.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..0c1ca04 100644
--- a/contracts/ERC20MultiDelegateOrig.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -1,8 +1,6 @@
 // SPDX-License-Identifier: MIT
 pragma solidity ^0.8.2;
 
-import {Address} from "@openzeppelin/contracts/utils/Address.sol";
-
 import "@openzeppelin/contracts/access/Ownable.sol";
 import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
 import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
@@ -23,8 +21,6 @@ contract ERC20ProxyDelegator {
  * @dev A utility contract to let delegators to pick multiple delegate
  */
 contract ERC20MultiDelegate is ERC1155, Ownable {
-    using Address for address;
-
     ERC20Votes public token;
 
     /** ### EVENTS ### */
```

## [N-02] Unnecessary getBalanceForDelegate() function

The `getBalanceForDelegate()` function is an `internal` function, and is only called in the `_processDelegation()` function to verify that the `msg.sender` is entitled to transfer delegation from a `source` (line 131).

```solidity
File: contracts/ERC20MultiDelegate.sol
124:     function _processDelegation(
125:         address source,
126:         address target,
127:         uint256 amount
128:     ) internal {
129:         uint256 balance = getBalanceForDelegate(source);
130: 
131:         assert(amount <= balance);
132: 
133:         deployProxyDelegatorIfNeeded(target);
134:         transferBetweenDelegators(source, target, amount);
135: 
136:         emit DelegationProcessed(source, target, amount);
137:     }

192:     function getBalanceForDelegate(
193:         address delegate
194:     ) internal view returns (uint256) {
195:         return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
196:     }
```

The `_processDelegation()` function, in turn, is only called in the `_delegateMulti()` function (line 100).

```solidity
File: contracts/ERC20MultiDelegate.sol
98:             if (transferIndex < Math.min(sourcesLength, targetsLength)) {
99:                 // Process the delegation transfer between the current source and target delegate pair.
100:                 _processDelegation(source, target, amount);
101:             } else if (transferIndex < sourcesLength) {
102:                 // Handle any remaining source amounts after the transfer process.
103:                 _reimburse(source, amount);
104:             } else if (transferIndex < targetsLength) {
105:                 // Handle any remaining target amounts after the transfer process.
106:                 createProxyDelegatorAndTransfer(target, amount);
107:             }
108:         }
109: 
110:         if (sourcesLength > 0) {
111:             _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
112:         }
113:         if (targetsLength > 0) {
114:             _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
115:         }
116:     }
```
A check of ERC1155 token ownership by `msg.sender`, however, necessarily occurs again when the source tokens are burnt in the `_delegateMulti()` function, in the Openzeppelin hook (line 383).

```solidity
File: @openzeppelin/contracts/token/ERC1155/ERC1155.sol
378:         for (uint256 i = 0; i < ids.length; i++) {
379:             uint256 id = ids[i];
380:             uint256 amount = amounts[i];
381: 
382:             uint256 fromBalance = _balances[id][from];
383:             require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");
384:             unchecked {
385:                 _balances[id][from] = fromBalance - amount;
386:             }
387:         }
```

All of this means that the `getBalanceForDelegate()` function is unnecessary, as the check it performs necessarily  occurs later the only time it is called.

### Recommended Mitigation

Consider removing the `getBalanceForDelegate()` function.

```diff
diff --git a/contracts/ERC20MultiDelegateOrig.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..fa22745 100644
--- a/contracts/ERC20MultiDelegateOrig.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -126,10 +126,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address target,
         uint256 amount
     ) internal {
-        uint256 balance = getBalanceForDelegate(source);
-
-        assert(amount <= balance);
-
         deployProxyDelegatorIfNeeded(target);
         transferBetweenDelegators(source, target, amount);
 
@@ -189,12 +185,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         return proxyAddress;
     }
 
-    function getBalanceForDelegate(
-        address delegate
-    ) internal view returns (uint256) {
-        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
-    }
-
     function retrieveProxyContractAddress(
         ERC20Votes _token,
         address _delegate
```

## [N-03] Unnecessary input variables

The `_token` input variable in the `retrieveProxyContractAddress()` function is a global state variable and can be accessed without needing an input variable.

```solidity
File: contracts/ERC20MultiDelegate.sol
198:     function retrieveProxyContractAddress(
199:         ERC20Votes _token,
200:         address _delegate
201:     ) private view returns (address) {
```

### Recommended Mitigation

Consider removing the `_token` input variable.

```diff
diff --git a/contracts/ERC20MultiDelegateOrig.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..0b142a8 100644
--- a/contracts/ERC20MultiDelegateOrig.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -144,7 +144,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
     function _reimburse(address source, uint256 amount) internal {
         // Transfer the remaining source amount or the full source amount
         // (if no remaining amount) to the delegator
-        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
+        address proxyAddressFrom = retrieveProxyContractAddress(source);
         token.transferFrom(proxyAddressFrom, msg.sender, amount);
     }
 
@@ -165,15 +165,15 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address to,
         uint256 amount
     ) internal {
-        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-        address proxyAddressTo = retrieveProxyContractAddress(token, to);
+        address proxyAddressFrom = retrieveProxyContractAddress(from);
+        address proxyAddressTo = retrieveProxyContractAddress(to);
         token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
     }
 
     function deployProxyDelegatorIfNeeded(
         address delegate
     ) internal returns (address) {
-        address proxyAddress = retrieveProxyContractAddress(token, delegate);
+        address proxyAddress = retrieveProxyContractAddress(delegate);
 
         // check if the proxy contract has already been deployed
         uint bytecodeSize;
@@ -196,12 +196,11 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
     }
 
     function retrieveProxyContractAddress(
-        ERC20Votes _token,
         address _delegate
     ) private view returns (address) {
         bytes memory bytecode = abi.encodePacked(
             type(ERC20ProxyDelegator).creationCode, 
-            abi.encode(_token, _delegate)
+            abi.encode(token, _delegate)
         );
         bytes32 hash = keccak256(
             abi.encodePacked(
```