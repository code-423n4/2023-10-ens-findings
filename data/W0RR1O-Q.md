[L-01] Replace inline assembly with account.code.length
========================================================

`<address>.code.length` can be used in Solidity >= 0.8.0 to access an account’s code size and check if it is a contract without inline assembly.

vulnerable code:
* https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol?#L179-L182

Suggestion:
```
return proxyAddress.code.length != 0;

```

