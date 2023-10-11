### [L-01] Passing `address(0)` inside `sources/targets`.
It's good practice to always have a `address(0)` check when working with address'.

### [NC-01] It's possible to call `_processDelegation` with the same `source` and `target` address.
This does nothing except waste the caller's gas.

### [NC-02] Passing 0 inside `amounts`.
It's general good practice to verify that when transferring tokens, `amount` is > 0.

