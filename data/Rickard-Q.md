# [L-01] Floating pragma
## Lines of code
[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2)
## Impact
Contracts should be deployed with the same compiler version and flags that they have been tested thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.       
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103).
## Proof of Concept
````solidity
1    // SPDX-License-Identifier: MIT
2:   pragma solidity ^0.8.2;
3
````
[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2)
## Tools Used
Manual Review
## Recommended Mitigation Steps
Lock the pragma version and also consider known bugs ([https://github.com/ethereum/solidity/releases](https://github.com/ethereum/solidity/releases)) for the compiler version that is chosen.