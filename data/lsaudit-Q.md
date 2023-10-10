# [QA-1] `_reimburse`, function does not emit event
Function `_reimburse`, which reimburses any remaining source amounts back to the delegator after the delegation transfer process does not emit event

```
function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

# [N-1] `constructor` in `ERC20ProxyDelegator` misses NatSpec
It is recommended that Solidity contracts are fully annotated using NatSpec. While this issue has been already reported by the bot-race, the bot-race did not mention missing NatSpec for the constructor.

There's no a single line of comment which explains each of the parameter used in the `constructor`:

```
 constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
```

For the reference - please check how the `constructor` is described in `ERC20MultiDelegate`


# [N-2] Incorrect comment in `_reimburse`

```
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

This comment should be moved before the `_reimburse` function call. The current implementation of `_reimburse` transfers `amount` which is passed by the parameter. Saying that it transfers either the remaining source amount of the full source amount is misleading, since this function transfers the whole `amount` which was passed by the parameter.

# [N-3] Protocol does not provide any documentation
Having all functions and features of the contract well documented helps during the audit-process. There wasn't any provided documentation on the Code4rena contest page and on the protocols website.  

# [N-4] `uint256` mixed with `uint`
While `uint256` is basically the same as `uint`, it's good coding practice to stick to using one version across the whole code-base. The more recommended one is `uint256`, since it explicitly states the size of the datatype.

The code-base, in most cases uses `uint256`, however, in two lines, it uses `uint`:

```
86:         uint transferIndex = 0;
179:        uint bytecodeSize;
```