## FLOATING PRAGMA
The project contains an instance of floating pragma. Contracts should
be deployed with the same compiler version and flags that they have
been tested thoroughly. Locking the pragma helps to ensure that
contracts do not accidentally get deployed using, for example, either
an outdated compiler version that might introduce bugs that affect the
contract system negatively or a pragma version too recent which has not
been extensively tested.

Code- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2 
      pragma solidity ^0.8.2;

Recommendation:
Consider locking the pragma version with known bugs for the compiler
version by removing the caret (^) symbol. When possible,