1.use require instead of assert
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131

Contracts use assert() instead of require() in multiple places. This causes a Panic error on failure and prevents the use of error strings.

Prior to solc 0.8.0, assert() used the invalid opcode which used up all the remaining gas while require() used the revert opcode which refunded the gas and therefore the importance of using require() instead of assert() was greater. However, after 0.8.0, assert() uses revert opcode just like require() but creates a Panic(uint256) error instead of Error(string) created by require(). Nevertheless, Solidity’s documentation says:

"Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.”