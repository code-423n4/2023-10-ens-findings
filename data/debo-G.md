## [G-01] abi.encode is not as cheap as abi.encodepacked in the ERC20MultiDelegate contract
**Description**
The function retrieveProxyContractAddress is using the abi.encode formula.
abi.encode is like the CHAR data type which is fixed as using maximum length storage even if you do not fill it. Whilst abi.encodePacked is like VARCHAR and only uses the length populated.
**Mitigation**
Utilize abi.encodePacked() not abi.encode() because it uses less storage hence being cheaper.
**Location**
```txt
contracts/ERC20MultiDelegate.sol=> Line: 204
```
**Vulnerable code snippet**
```sol
// Line: 204
            abi.encode(_token, _delegate)
```

## [G-02] Storage variable caching in memory for ERC20MultiDelegate contract
**Description**
The state variable called token is being utilised many times in the functions called transferBetweenDelegators & deployProxyDelegatorIfNeeded & _reimburse.
Storage loads can be costly such as 100 gas from the first call in comparison to MLOAD/MSTORE of 3 gas per call.
**Mitigation**
Cache within memory rather than Storage variables called over and over again within the function. 
 This will charge 1 SLOAD and not call over and over again being a fraction of the cost.
**Locations**
```sol
// Line 28
    ERC20Votes public token;
```
```sol
// Line 144-149
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```
```sol
// Line 163-171
    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```
```sol
// Line 173-190
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
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```