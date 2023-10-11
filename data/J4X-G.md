## [GAS-01] Unnecessary call to Math.max
[Line 87](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87)

**Issue Description:**
The call to `Math.max` is redundant since `amountsLength` is already defined as `Math.max(sourcesLength, targetsLength)`.

**Recommended Mitigation:**
The function call can be replaced with the constant `amountsLength`.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..f2839b8 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -84,7 +84,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < amountsLength;
             transferIndex++
         ) {
             address source = transferIndex < sourcesLength
```

**Gas Reductions:**
Gas savings (using testsuite): 308

---
## [GAS-02] `retrieveProxyContractAddress()` does not need the parameter `_token`
[Line 199](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L199)

**Issue Description:**
The function `retrieveProxyContractAddress()` gets passed the parameter `_token`, which is not needed as it always has to be the already declared variable `token`.

**Recommended Mitigation:**
Remove the parameter `_token` and use the storage variable `token` instead.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..0b142a8 100644
--- a/contracts/ERC20MultiDelegate.original
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

**Gas Reductions:**
Gas savings (using testsuite):  20

---
## [GAS-03] Unnecessary check on balance in `_processDelegation`
[Line 131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131)

**Issue Description:**
The function `_processDelegation()` implements a check to verify if the user has delegated enough to the delegate from whom they want to transfer now. This check is not needed, as the amount of tokens gets burned later on, and the requirement `require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");` from the ERC1155 library will trigger.

**Recommended Mitigation:**
Remove the check.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..ac612fd 100644
--- a/contracts/ERC20MultiDelegate.original
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
 
```


In the current testsuite, this change will lead to the error 'ERC20: insufficient allowance' being thrown in the testcase 'should revert if at least one source address is not a delegate of the caller' due to there not being any tokens in Charlie's proxy. In reality, this can be the case, so this ERC20 require will not stop someone from withdrawing. To showcase that in a real-world scenario where other users' tokens can also be stored in the wrongfully chosen delegatee's proxy, I wrote a simple testcase to verify that the intended requirement from the ERC1155 library triggers on wrong use.

```js
it('should revert if at least one source address is not delegate of the caller', async () => {
	var delegatorTokenAmount = await token.balanceOf(deployer);
	const [_, secondDelegator] = await ethers.getSigners();
	
	await token.transfer(secondDelegator.address, delegatorTokenAmount.sub(100));
	
	delegatorTokenAmount = await token.balanceOf(deployer);
	const secondAmount = await token.balanceOf(secondDelegator.address);
	
	// give allowance to multi delegate contract
	await token.approve(multiDelegate.address, delegatorTokenAmount);
	await token.connect(secondDelegator).approve(multiDelegate.address, secondAmount);
	
	await multiDelegate.delegateMulti([], [bob], [delegatorTokenAmount]);
	await multiDelegate.connect(secondDelegator).delegateMulti([], [charlie], [secondAmount]);
	
	await expect(
	multiDelegate.delegateMulti([charlie], [dave], [delegatorTokenAmount])
	).to.be.revertedWith(
	'ERC1155: burn amount exceeds balance'
	);
});
```

**Gas Reductions:**
Gas savings (using testsuite):  1140

---
## [GAS-04] Function get `getBalanceForDelegate()` is not needed
[Line 199](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L199)

**Issue Description:**
The function `getBalanceForDelegate()` is just a wrapper for the underlying `balanceOf()` function. As it is just internal, this wastes gas and does not bring a benefit to users.

**Recommended Mitigation:**
Remove the function `getBalanceForDelegate()` and use the function `balanceOf()` instead.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..86075ea 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -126,7 +126,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address target,
         uint256 amount
     ) internal {
-        uint256 balance = getBalanceForDelegate(source);
+        uint256 balance = balanceOf(msg.sender, uint256(uint160(source)));
 
         assert(amount <= balance);
 
@@ -188,13 +188,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         }
         return proxyAddress;
     }
-
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

**Gas Reductions:**
Gas savings (using testsuite): 931
Deployment Gas savings (using testsuite): 58098

---
## [GAS-05] Function get `getBalanceForDelegate()` is not needed
[Line 100](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L100)

**Issue Description:**
In the function `_delegateMulti()`, the function `Math.min()` gets called in every for loop iteration. This is unnecessary, as both values that are compared do not change.

**Recommended Mitigation:**
Do the call to `Math.min()` and save the value in memory, comparing to the memory value from then on.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..2b6f964 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -81,6 +81,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );
 
+        uint256 minLength = Math.min(sourcesLength, targetsLength);
+
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
@@ -95,7 +97,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 : address(0);
             uint256 amount = amounts[transferIndex];
 
-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex < minLength) {
                 // Process the delegation transfer between the current source and target delegate pair.
                 _processDelegation(source, target, amount);
             } else if (transferIndex < sourcesLength) {
```

**Gas Reductions:**
Gas savings (using testsuite): 124

---
## [GAS-06] Non needed function `delegateMulti()`
[Line 57](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57)

**Issue Description:**
The external function `delegateMulti()` is just a wrapper for the internal function `_delegateMulti()`. As there are no other calls to `_delegateMulti()`, the function adds unnecessary overhead.

**Recommended Mitigation:**
Inline `_delegateMulti()` into `delegateMulti()`.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..f4183b7 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -59,14 +59,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256[] calldata targets,
         uint256[] calldata amounts
     ) external {
-        _delegateMulti(sources, targets, amounts);
-    }
-
-    function _delegateMulti(
-        uint256[] calldata sources,
-        uint256[] calldata targets,
-        uint256[] calldata amounts
-    ) internal {
         uint256 sourcesLength = sources.length;
         uint256 targetsLength = targets.length;
         uint256 amountsLength = amounts.length;
```

**Gas Reductions:**
Gas savings (using testsuite):  54
Deployment Gas savings (using testsuite): 5172

---
## [GAS-07] Newly declaring memory variables in every loop wastes gas
[Line 94](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L94)
[Line 97](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L97)
[Line 100](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L100)

**Issue Description:**
n the loop in the function `_delegateMulti()`, the variables `source`, `target`, and `amount` get newly declared with each iteration, wasting gas.

**Recommended Mitigation:**
Declare the variables once before the loop and then overwrite them at each iteration. Since all of them get overwritten at each iteration, there is no danger of an older value being reused.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..5ad328c 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -81,19 +81,23 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );
 
+        address source;
+        address target;
+        uint256 amount;
+
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
             transferIndex < Math.max(sourcesLength, targetsLength);
             transferIndex++
         ) {
-            address source = transferIndex < sourcesLength
+            source = transferIndex < sourcesLength
                 ? address(uint160(sources[transferIndex]))
                 : address(0);
-            address target = transferIndex < targetsLength
+            target = transferIndex < targetsLength
                 ? address(uint160(targets[transferIndex]))
                 : address(0);
-            uint256 amount = amounts[transferIndex];
+            amount = amounts[transferIndex];
 
             if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                 // Process the delegation transfer between the current source and target delegate pair.
```

**Gas Reductions:**
Gas savings (using testsuite): 19

---
## [GAS-08] amounts.length can be accessed after the requirement
[Line 72](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L72)

**Issue Description:**
In the function `_delegateMulti()`, the length of all three arrays gets accessed. As there is a require statement on the sources and targets length, which can fail, the `amounts` array's length should be accessed after the require, as it is not needed before and spends unnecessary gas in case of failure.

**Recommended Mitigation:**
Move the declaration and access after the require statement.

```diff
diff --git a/contracts/ERC20MultiDelegate.original b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..ae82a6b 100644
--- a/contracts/ERC20MultiDelegate.original
+++ b/contracts/ERC20MultiDelegate.sol
@@ -69,13 +69,14 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
     ) internal {
         uint256 sourcesLength = sources.length;
         uint256 targetsLength = targets.length;
-        uint256 amountsLength = amounts.length;
 
         require(
             sourcesLength > 0 || targetsLength > 0,
             "Delegate: You should provide at least one source or one target delegate"
         );
 
+        uint256 amountsLength = amounts.length;
+
         require(
             Math.max(sourcesLength, targetsLength) == amountsLength,
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
```

**Gas Reductions:**
Gas savings (using testsuite): undefined as no testcase for this situation is available
