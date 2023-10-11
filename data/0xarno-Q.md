## [L-01] Make `ERC20Votes public token` immutable in `ERC20MultiDelegate` contract!
```diff
- ERC20Votes public token;
+ ERC20Votes public immutable token;
```
### Lines of code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L28

## [L-02] `delegateMulti()` missing if sources[id] and targets[id] and amounts[id] is Zero

```diff
if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
              +  require(source != address(0) && target != address(0),"Source address cannot be zero");
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
              +  require(source != address(0), "Source address cannot be zero");
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
              +  require(target != address(0), "Source address cannot be zero");
                createProxyDelegatorAndTransfer(target, amount);
            }
```
### Lines of code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L109C1

## [L-03] Inadequate Validation for Addresses Exceeding `type(uint160).max` in `delegateMulti()`

### Description:

The `delegateMulti()` function processes delegations from source to `target` delegates using addresses provided in the sources and targets arrays by casting target[id] and source[id] to address(uint160(...)). However, it lacks a check to ensure that the addresses in the targets array are not greater than `type(uint160).max`. When `target[id]` exceeds `type(uint160).max`, the truncation of the address can result in incorrect and potentially unsafe addresses.

### Lines of code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L109


## [N-01] delete `assert(amount <= balance);`

### Description:
The `assert(amount <= balance);` statement in the `_processDelegation` function is currently used to check if the amount being delegated is less than or equal to the balance of the source delegate. While this check helps ensure that delegations do not exceed available funds, it has been determined that it is not essential for the intended functionality of the contract and can be safely removed.
### Lines of code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131