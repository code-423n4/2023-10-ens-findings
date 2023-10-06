# 1. Using require instead of assert

## Description

In the `_processDelegation` function, the code uses the `assert` statement for error handling. While `assert` is suitable for checking internal conditions that should never occur if the contract is functioning correctly, it's not the best choice for handling user-triggered errors or conditions that can arise due to external factors.

https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L131

## Remediation

To improve error handling and provide more meaningful feedback to users, it's recommended to replace the `assert` statement with a `require` statement that includes an informative error message. This message should explain why the condition failed and what the user needs to do to rectify it.