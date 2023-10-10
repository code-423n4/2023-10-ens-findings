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