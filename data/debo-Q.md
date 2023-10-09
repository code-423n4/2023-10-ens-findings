## [L-01] ERROR-PRONE TYPECASTING in the ERC20MultiDelegate contract
**Description**

The variable called delegate is double casted from uint160 to uint256.
The result is unstable code.

The variable called hash is double casted from uint256 to uint160.
The result is unstable code.

**Locations**
```txt
contracts/ERC20MultiDelegate.sol#L195-L195
```
```txt
contracts/ERC20MultiDelegate.sol#L214-L214
```
**Vulnerable code snippets**
```sol
// Line 195
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```
```sol
// Line 214
        return address(uint160(uint256(hash)));
```
**Mitigation**
Use OpenZeppelin safeCast.

## [L-02] USE OWNABLE2STEP in the ERC20MultiDelegate contract
**Description**
The Ownable2Step is more safe than Ownable.
Because unsafe tranfers of ownership are not possible.
The new owner will have to acknowledge ownership now which adds an extra layer of security.
**Vulnerable code snippet**
```sol
// Line 25
contract ERC20MultiDelegate is ERC1155, Ownable {
```
**Reference**
```txt
contracts/ERC20MultiDelegate.sol#L25-L216
```
**Mitigation**
Use Ownable2Step or Ownable2StepUpgradeable based on business logic.

## [L-03] Missing Events in the ERC20MultiDelegate contract
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