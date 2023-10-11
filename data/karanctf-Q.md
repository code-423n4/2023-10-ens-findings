## Create a mapping for saving Proxy Contract addresses
Saving addresses in mapping with make retrieving them easier and efficient
**summary:** The function `retrieveProxyContractAddress()` is calculating address everytime transfer happens. Instead of calculating it again and again make a mapping which saves previously computed addresses
```diff
++ mapping (address=>address) savedProxy;

function deployProxyDelegatorIfNeeded(

address delegate

) internal returns (address) {


++ if(savedProxy[delegate] != address(0)) return savedProxy[delegate];

address proxyAddress = retrieveProxyContractAddress(token, delegate);

++ savedProxy[delegate] = proxyAddress;

// check if the proxy contract has already been deployed

uint bytecodeSize;

assembly {

bytecodeSize := extcodesize(proxyAddress)

}

  

// if the proxy contract has not been deployed, deploy it

if (bytecodeSize == 0) {

new ERC20ProxyDelegator{salt: 0}(token, delegate);

emit ProxyDeployed(delegate, proxyAddress);

}

return proxyAddress;

}
```






## Internal and private functions should have an underscore prefix.
[ref](https://primitivefi.notion.site/Solidity-Style-44daebebfbd645b0b9cbad7075ba42fe)
add a \_ prefix on internal and private functions
Instances:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L198
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173
