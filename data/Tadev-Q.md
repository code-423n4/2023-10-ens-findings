## [L‑01] Missing checks in `_delegateMulti` function for 0 values in `amounts` array 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L68

Internal `_delegateMulti` function should check that `uint256[] calldata amounts` array doesn't contain any value equal to 0. If it does, it will finally result in calling `transferfrom` with amount equal to zero, which is not a desired behavior, as it will consume gas for no action.

Adding a check in the for loop could prevent executing `transferfrom` with 0 amount, and continue to the next iteration of the loop.

For example, we could addd the check after this line : https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L96

```
if (amount == 0) {
  continue;
}
