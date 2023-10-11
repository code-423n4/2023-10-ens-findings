# 1. Floating pragma version should not be used

Do not use the solidity floating version, instead use the latest stable version. Here I suggest using version 0.8.19.

# 2. Do not use underscore hyphens for internal function names

In your code, I understand that you need to use underscore hyphens for the internal function, but here you don't do that.

## Proof Of Concept

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L197

```
   function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
       ...
    }

    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        ...
    }

    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        ...
    }

    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        ...
    }
```

## Recommendation

```
   function _createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
       ...
    }

    function _transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        ...
    }

    function _deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        ...
    }

    function _getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        ...
    }
```

# 3. Correct the way to import files

You use two different ways of importing files in the same contract. This is untidy and disruptive. In addition, you should also separate the ERC20ProxyDelegator contract into a separate file.

## Proof Of Concept

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L3C1-L10

```
import {Address} from "@openzeppelin/contracts/utils/Address.sol";

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
```

## Recommendation

```
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {ERC1155} from "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import {ERC20Votes} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import {Math} "@openzeppelin/contracts/utils/math/Math.sol";
```