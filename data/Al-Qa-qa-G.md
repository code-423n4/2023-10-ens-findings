# Declare repeatable computed variables in the for loop.

Store the computed variables that are being used in for loop, and don't compute them each iteration.
Using Math library functions such as `Math.min` or `Math.max` in each loop iteration will consume much gas. Declare the variables needed in for loop comparison outside the for loop, and use them instead.

There are 2 instances:

- ERC20MultiDelegate.sol ([#L80](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L80), [#L87](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L87))
- ERC20MultiDelegate.sol ([#L98](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L98))


