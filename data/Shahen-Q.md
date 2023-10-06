### Use `create2` method for deploying proxy contracts.

In function `deployProxyDelegatorIfNeeded` uses `new` method to deploy proxy contract, But `create2` opcode method is more compatible with future EVM upgrades.

#### Code Snippet:

https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L186