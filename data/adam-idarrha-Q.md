## Summary<a name="Summary">

### Low Issues
| |Issue|number of Instances
|-|:-|:-|
| [L&#x2011;01](#L&#x2011;01) | No Need to call function `retrieveProxyContractAddress` because function `deployProxyDelegatorIfNeeded` already gives the proxy address | 1 
| [L&#x2011;02](#L&#x2011;02) | the function `delegateMulti` does an unsafe external call back to the caller, so it should have reentrency guard | 1 

### Non Critical Issues
| |Issue|number of Instances
|-|:-|:-|
| [N&#x2011;01](#N&#x2011;01) |  | 1 

## Low Issues:

### <a href="#Summary">[L&#x2011;01]</a><a name="L&#x2011;01"> No Need to call function `retrieveProxyContractAddress` because function `deployProxyDelegatorIfNeeded` already gives the proxy address. 

in the function `_processDelegation` we call `deployProxyDelegatorIfNeeded` but don't store the address where the proxy is deployed:

- *ERC20MultiDelegate.sol* ( [#L133](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L133)):
    
```solidity=133
deployProxyDelegatorIfNeeded(target);
transferBetweenDelegators(source, target, amount);
```

but instead we call `retrieveProxyContractAddress` again to get the proxy address, which is a waste since we incur all the gas overhead from doing the calculations again, and also affects readability:

- *ERC20MultiDelegate.sol* ( [#L169](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L169)):
    
```solidity=169
address proxyAddressTo = retrieveProxyContractAddress(token, to);
```
    
**recommendation:**
    
in the function `transferBetweenDelegators` we should call `deployProxyDelegatorIfNeeded` and store the address like so:

```solidity
address proxyAddressTo = deployProxyDelegatorIfNeeded(target);
```

### <a href="#Summary">[L&#x2011;02]</a><a name="L&#x2011;02"> the function `delegateMulti` does an unsafe external call back to the caller, so it should have reentrency guard
    
the function `_mintBatch` does an unsafe external calls back to the address specified in the first parameter, which is `msg.sender` in this case, if it's a contract:
    
- *ERC1155.sol* ( [#L468](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/e1b3d8c7ee2c97868f4ab107fe8b7e19f0a8db9f/contracts/token/ERC1155/ERC1155.sol#L468-L479)):
    
```solidity=468
function _doSafeBatchTransferAcceptanceCheck(
        address operator,
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) private {
        if (to.isContract()) {
            try IERC1155Receiver(to).onERC1155BatchReceived(operator, from, ids, amounts, data) returns (
                bytes4 response
            )
```

this function is called by `_mintbatch` , and it calls back to the `to` address, and in the contract we set it to `msg.sender`.
    
- *ERC20MultiDelegate.sol* ( [#L114](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L114)):
    
```solidity=114
_mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```

**recommendation**:

because this function does an unsafe external call back to the caller, consider adding reentrency guard to this function.
    
## Non Critical Issues:

### <a href="#Summary">[N&#x2011;01]</a><a name="N&#x2011;01"> redundunt check for balance of user in `_processDelegation`
    
the function `_processDelegation` checks for the balance of the user before transfering tokens between the two proxies, but the transaction will fail inevitably if the user had insufficient balance for the transfer because of the `_burnbatch` call.
    
- *ERC20MultiDelegate.sol* ( [#L114](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131)):
    
```solidity=129
uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);
```