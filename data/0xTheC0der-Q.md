## Low 1: The `delegateMulti(...)` method allows user-defined target IDs that cannot be re-delegated
Users are free to specify delegation sources/targets as `uint256`, see [signature of delegateMulti(...)](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L58-L59). Those sources/targets are actually ERC1155 token IDs, which track ownership of delegations, and are burned in [L111](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L111) / minted in [L114](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L114).  
However, those delegation sources/targets actually refer to addresses and are down-casted from `uint256` (256 bits) to `address` (160 bits) in [L91](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L91) and [L94](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L94) which leads to the following impacts:
* Users can arbitrarily choose the upper 96 bits of target IDs referring to the **same** underlying delegation target address. This ID then has to be used as source ID for re-delegation and withdrawal due to the underlying ERC1155 token.
* Such user-defined IDs lead to **invisible delegation** since [getBalanceForDelegate(address delegate)](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L192-L196) internally up-casts the given address to an ID. Therefore, this method doesn't work with those IDs having more than 160 bits.
* Re-delegation, see [_processDelegation(...)](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L118-L137), will **always fail** due to the assertion in [L131](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131) that doesn't account for invisible delegation with such user-defined IDs.  

The following PoC modifies an exisiting test case in order to demonstrate the use of custom IDs to cause invisible delegation, which subsequently leads to re-delegation failure:
```diff
diff --git a/test/delegatemulti.js b/test/delegatemulti.js
index e557d4e..88d6ff6 100644
--- a/test/delegatemulti.js
+++ b/test/delegatemulti.js
@@ -236,14 +236,16 @@ describe('ENS Multi Delegate', () => {
   });
 
   describe('re-deposit', () => {
-    it('should be able to re-delegate multiple delegates in behalf of user (1:1)', async () => {
+    it.only('should not be able to re-delegate multiple delegates in behalf of user (1:1)', async () => {
       const delegatorTokenAmount = await token.balanceOf(deployer);
       // const customAmount = ethers.utils.parseEther('10000000.0'); // ens
 
+      const numberOverAddressRange = ethers.BigNumber.from(2).pow(160); // 2^160
+
       // give allowance to multi delegate contract
       await token.approve(multiDelegate.address, delegatorTokenAmount);
-      // delegate multiple delegates
-      const delegates = [deployer, alice];
+      // delegate multiple delegates with custom ERC1155 id > address range
+      const delegates = [deployer, ethers.BigNumber.from(alice).or(numberOverAddressRange)];
 
       const amounts = delegates.map(() =>
         delegatorTokenAmount.div(delegates.length)
@@ -272,8 +274,10 @@ describe('ENS Multi Delegate', () => {
         delegatorTokenAmount.div(newDelegates.length)
       );
 
-      await multiDelegate.delegateMulti(delegates, newDelegates, newAmounts);
+      // triggers assert(amount <= balance) in _processDelegation(...)
+      await expect(multiDelegate.delegateMulti(delegates, newDelegates, newAmounts)).to.be.reverted;
 
+      /*
       for (let delegateTokenId of delegates) {
         let balance = await multiDelegate.balanceOf(deployer, delegateTokenId);
         expect(balance.toString()).to.equal('0');
@@ -291,6 +295,7 @@ describe('ENS Multi Delegate', () => {
       expect(votesOfNewDelegate.toString()).to.equal(
         delegatorTokenAmount.div(newDelegates.length).toString()
       );
+      */
     });
 
     it('should be able to re-delegate multiple delegates in behalf of user (many:many)', async () => {

```
I recommend to consistently use the `address` type, especially in the [signature of delegateMulti(...)](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L58-L59), to completely resolve this issue. As a result, the unsafe down-casts can be removed and only internal up-casts from addresses to ERC1155 token IDs are necessary.
