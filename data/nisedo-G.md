# G-01 Switch OZ to Solmate

Solmate's implementations of the libraries used in the protocol are more optimized, saving gas for the user.


[ERC20MultiDelegate.sol#L4-L9:](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4-L9)
```solidity
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
```