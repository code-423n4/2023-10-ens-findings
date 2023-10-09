## [L-01] Missing Events in the ERC20MultiDelegate contract
**Locations**
```sol
contracts/ERC20MultiDelegate.sol#L144-L149
```
```sol
contracts/ERC20MultiDelegate.sol#L151-L153
```
```sol
contracts/ERC20MultiDelegate.sol#L163-L171
```
**Description**
Use emit events after each function as a way to log transactions.
This will help to rapidly respond to issues or breaches.
**Vulnerable Functions**
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
// Line 151-153
    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
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
**Mitigation**
Use emit functions.

## [L-02] Floating pragmas and outdated version in the ERC20MultiDelegate contract
**Description**
Floating pragmas can introduce security vulnerabilities.
Because you can cross-compile with different versions.
**Location**
```txt
/contracts/ERC20MultiDelegate.sol - ^0.8.2
```
**Mitigation**
Use `0.8.18`.