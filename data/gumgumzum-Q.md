### Downcasting input `targets` and `sources` to uint160 in `_delegateMulti` can lead to different `ERC20MultiDelegate` token ids for the same delegate 
### The `assert` in `_processDelegation` can fail by trying to transfer voting power between delegates for which the user doesn't hold enough `ERC20MultiDelegate` tokens
### `Address` is not used in `ERC20MultiDelegate`