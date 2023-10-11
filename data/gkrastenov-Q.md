## NC
# [NC-01] Use amountsLength instead of Math.max(sourcesLength, targetsLength)
In the `_delegateMulti` function, it is necessary for `Math.max(sourcesLength, targetsLength)` to equal `amountsLength`. Therefore, instead of using `Math.max(sourcesLength, targetsLength)` in the for loop, you can directly use the `amountsLength` variable.

# [NC-02] Do not call retrieveProxyContractAddress function twice
Do not call the `retrieveProxyContractAddress` function twice. 

Instead of using `deployProxyDelegatorIfNeeded(target)` within the `_processDelegation` function, it can be called directly in the `transferBetweenDelegators` function, replacing address` proxyAddressTo = retrieveProxyContractAddress(token, to)`. The `deployProxyDelegatorIfNeeded` function will return the proxyAddress directly, eliminating the need to call `retrieveProxyContractAddress`.

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


function transferBetweenDelegators( 
        address from,
        address to,
        uint256 amount
    ) internal {
       address proxyAddressFrom = retrieveProxyContractAddress(token, from);
+       address proxyAddressTo = deployProxyDelegatorIfNeeded(to);
-       address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```

# [NC-03] All internal functions should start with _ prefix
All internal functions should start with `_` prefix. This will increase code readability.