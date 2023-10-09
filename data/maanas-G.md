### Summary

|ID|Title|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [GAS-01](#gas-01-avoid-unwanted-assert)| Avoid unwanted assert | 1 | 2164 |
| [GAS-02](#gas-02-change-deployment-to-use-salt)| Change deployment to use salt | 1 | 630 |


Total: 2 instances over 2 issues with 2794 gas saved

## Gas Optimizations

### [GAS-01] Avoid unwanted assert
Assert can be avoided as the `burn` present will revert in case the user doesn't have enough balance. The assert function also doesn't always assert that the user has enough balance since if same source is passed multiple times, the check could be ineffective

*There are 1 instances of this issue*

```solidity
File: contracts/ERC20MultiDelegate.sol

129:                uint256 balance = getBalanceForDelegate(source);
130:        
131:                assert(amount <= balance); // assert and the previous fetch of user's balance can be avoided

```

*GitHub*: [129](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L129-L131)


### [GAS-02] Change deployment to use salt
Changing from using constructor arguments to a salt and callback function allows to use a fixed hash when computing the proxy address. But this does increase the deployment cost by 44k which will be compensated atleast when there are 70 different delegator proxies

*There are 1 instances of this issue*

```diff
File: contracts/ERC20MultiDelegate.sol

diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..fad0798 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -13,12 +13,14 @@ import "@openzeppelin/contracts/utils/math/Math.sol";
  * This is a proxy delegator contract to vote given delegate on behalf of original delegator
  */
 contract ERC20ProxyDelegator {
-    constructor(ERC20Votes _token, address _delegate) {
+    constructor() {
+        (ERC20Votes _token, address _delegate) = ERC20MultiDelegate(msg.sender).deployCallBack();
         _token.approve(msg.sender, type(uint256).max);
         _token.delegate(_delegate);
     }
 }
 
+
 /**
  * @dev A utility contract to let delegators to pick multiple delegate
  */
@@ -46,8 +48,11 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         string memory _metadata_uri
     ) ERC1155(_metadata_uri) {
         token = _token;
+        CREATION_CODE_HASH = keccak256(type(ERC20ProxyDelegator).creationCode);    
     }
 
+    address deployDelegate;
+    bytes32 immutable CREATION_CODE_HASH;
     /**
      * @dev Executes the delegation transfer process for multiple source and target delegates.
      * @param sources The list of source delegates.
@@ -183,7 +188,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
 
         // if the proxy contract has not been deployed, deploy it
         if (bytecodeSize == 0) {
-            new ERC20ProxyDelegator{salt: 0}(token, delegate);
+            deployDelegate = delegate;
+            new ERC20ProxyDelegator{salt: bytes32(uint256(uint160(delegate)))}();
             emit ProxyDeployed(delegate, proxyAddress);
         }
         return proxyAddress;
@@ -199,18 +205,20 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         ERC20Votes _token,
         address _delegate
     ) private view returns (address) {
-        bytes memory bytecode = abi.encodePacked(
-            type(ERC20ProxyDelegator).creationCode, 
-            abi.encode(_token, _delegate)
-        );
         bytes32 hash = keccak256(
             abi.encodePacked(
                 bytes1(0xff),
                 address(this),
-                uint256(0), // salt
-                keccak256(bytecode)
+                uint256(uint160(_delegate)), // salt
+                CREATION_CODE_HASH
             )
         );
         return address(uint160(uint256(hash)));
     }
-}
+
+    function deployCallBack() external returns (ERC20Votes _token, address _delegate) {
+        _token = token;
+        _delegate = deployDelegate;
+        delete deployDelegate;
+    }
+}

```