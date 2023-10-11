# Summary

All gas optimizations were benchmarked using the test suite of the `ERC20MultiDelegate` contract, i.e. using the following config: `solc version 0.8.7`, `optimizer on`, and `200 runs`.

## [G-01] The `_processDelegation` function is wasting gas by making an unnecessary call to the `retrieveProxyContractAddress` function

**Approximate gas savings: ~2700 GAS per function call**

As we know, unlike when called by EOAs or other view functions, when view functions are called within another non-view functions, they actually incur gas fees. There is one place within the `ERC20MultiDelegate`, where we can make an optimization in order to remove one view function call within another non-view funciton.
You can find the lines of code under question + the diff that applies the optiization bellow:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L133

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L133

```diff
function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

-       deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
 }
```

```diff
function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-       address proxyAddressTo = retrieveProxyContractAddress(token, to);
+       address proxyAddressTo = deployProxyDelegatorIfNeeded(to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
}
```

## [G-02] Use != 0 or == 0 for unsigned integer comparison

**Approximate gas savings: ~6 GAS per function call**

For uint comparison in require() statements prior to solidity 0.8.13, it is cheaper to use != 0 or == 0 instead of > 0 or <= 0 for unsigned integer comparison.
The winning bot report already mentioned this optimizaiton, but it missed two places where it can be applied. Those are the following:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L110

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L113

```diff
-if (sourcesLength > 0) {
+if (sourcesLength != 0) {
    _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
}
-if (targetsLength > 0) {
+if (targetsLength != 0) {
    _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
}
```
