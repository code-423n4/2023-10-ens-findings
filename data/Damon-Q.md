## [L-01] Missing checks for `addrees(0)` in `targets` of function `_delegateMulti`
The function `_delegateMulti` does not check `address(0)` in `targets`. If the user call the function with `targets = [address(0)]`, the ENSToken would not distribute the corrosponding votes to `address(0)`;

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L82
