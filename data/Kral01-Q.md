## Lack of Check can cause loop to become unbounded

The [_delegateMulti](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65C4-L69C17) has no check to see if the amount is zero. This enables a person to feed large amounts of sources and targets while keeping the amounts to zero which can result in unbounded loop.

inside the loop,
```solidity
uint256 amount = amounts[transferIndex];
if(amount < 0){
    continue;
}
```

## Argument naming is confusing.

In most functions the argument is named as `delegate` while this is true for the most part, the term 'delegate' is used interchangeably throughout the contract. Suppose in case of a `_reimburse`, the call goes to `retrieveProxyContractAddress()` in which the choice `delegate` might not be the best choice.

## Used named imports.

Using named imports is a good coding convention that can increase the overall quality of the codebase.

```solidity
import {Address} from "@openzeppelin/utils/Address.sol";

import {Ownable} from "@openzeppelin/access/Ownable.sol";
import {ERC1155} from "@openzeppelin/token/ERC1155/ERC1155.sol";
import {ERC20Votes} from "@openzeppelin/token/ERC20/extensions/ERC20Votes.sol";
import {Math} from "@openzeppelin/utils/math/Math.sol";
```
## Internal function name can be prefixed with '_' .

Internal function names should be prefixed with '_' . This naming convention is followed in `_processDelegation` , `_reimburse` , `_delegateMulti` but not in other internal functions used in the contract.

```
function _createProxyDelegatorAndTransfer() internal {}
function _transferBetweenDelegators() internal {}
function _deployProxyDelegatorIfNeeded() internal {}
function _getBalanceForDelegate() internal {}
function _retrieveProxyContractAddress()internal  {}
```

