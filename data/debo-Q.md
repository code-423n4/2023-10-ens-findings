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
