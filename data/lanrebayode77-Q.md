## 1. Code Optimization
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L133
This call returns the address of the target contract but was not carched
``` solidity 
 deployProxyDelegatorIfNeeded(target);
```

In transferBetweenDelegators(), target contract address was computed again. Since,  transferBetweenDelegators() was only called from  _processDelegation(), the already computed target address should be passed instead of re-computing the address.

``` solidity
  function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        address proxyAddressTo = deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, proxyAddressTo, amount);

        emit DelegationProcessed(source, target, amount);
    }
```

``` solidity
function transferBetweenDelegators(
        address from,
        address proxyAddressTo,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```
Lesser lines of code, same implementation.
