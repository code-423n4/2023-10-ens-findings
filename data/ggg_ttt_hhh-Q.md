1. A lack of checks on `sources` and `targets` can result in tokens being blocked permanently.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L90-L92
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L111
While it's possible that this scenario won't occur, it's important to be cautious about input values exceeding `type(uint160).max` due to potential typos. 
For example, for a specific index `i`, `sources[i]` may be greater than `address(uint160(sources[i])`. In such cases, tokens get transferred to `address(uint160(sources[i]))`, but the balance of `sources[i]` will increase, and there might be no way to transfer these tokens from `address(uint160(sources[i]))`.

We can implement checks for `sources` and `targets` in `_delegateMulti` function.