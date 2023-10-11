- The function `deployProxyDelegatorIfNeeded` doesn’t validate deployed address.
    
    In the function `deployProxyDelegatorIfNeeded` there is no validation to check if the `deployedAddress` is equal to `proxyAddress`.
    
 ```solidity
    function deployProxyDelegatorIfNeeded(
            address delegate
        ) internal returns (address) {
            address proxyAddress = retrieveProxyContractAddress(token, delegate);
    
            // check if the proxy contract has already been deployed
            uint bytecodeSize;
            assembly {
                bytecodeSize := extcodesize(proxyAddress)
            }
    
            // if the proxy contract has not been deployed, deploy it
            if (bytecodeSize == 0) {
    -						new ERC20ProxyDelegator{salt: 0}(token, delegate);
    +           ERC20ProxyDelegator newDeployment = new ERC20ProxyDelegator{salt: 0}(token, delegate);
    +           require(address(newDeployment) == proxyAddress, "Address Mismatch");
                emit ProxyDeployed(delegate, proxyAddress);
            }
            return proxyAddress;
        }
```