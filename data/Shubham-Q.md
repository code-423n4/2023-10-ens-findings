## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Use `address(this)` instead of `this` | 1 |
| [NC-2](#NC-2) | Unnecessary import | 1 |

## [NC-1] Use `address(this)` instead of `this`

`this` and `address(this)` are the same things that have different terminologies due to these breaking changes.
`this` is the term used for the smart contract when the solidity version is below 0.5.0 .
`address(this)` is the term used in the latest versions of the solidity to refer to the smart contract.

```solidity
File: ERC20MultiDelegate.sol

195:   return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```
### Recommended
```solidity
File: ERC20MultiDelegate.sol

- 195:   return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+ 195:   return ERC1155(address(this)).balanceOf(msg.sender, uint256(uint160(delegate)));
```

## [NC-2] Unnecessary import

Some imports aren’t used inside the project

```solidity
File: ERC20MultiDelegate.sol

4:       import {Address} from "@openzeppelin/contracts/utils/Address.sol";
9:       import "@openzeppelin/contracts/utils/math/Math.sol";
```