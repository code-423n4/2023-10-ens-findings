# QA Report

##

## [L-1] Numbers downcast to `address`es may result in collisions

If a number is downcast to an `address` the upper bytes are truncated, which may mean that more than one value will map to the `address`.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

91:  ? address(uint160(sources[transferIndex]))
94: ? address(uint160(targets[transferIndex]))
214:  return address(uint160(uint256(hash)));

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L91-L94

##

## [L-2] ``require()`` should be used instead of ``assert()``	

``assert()`` is used to check for conditions that should never, ever be false. It's meant for detecting and protecting against invariants in your code. When an assert() statement evaluates to false, it indicates a bug in the code, and the entire transaction will be reverted. It ``does not return any gas``.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

131: assert(amount <= balance);

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131

### Recommended Mitigation
```diff
+ require(amount <= balance , "");

```
##

## [L-3] The default case is not implemented in ``_delegateMulti()`` function

A default case (often represented as an else or default statement) is used to handle situations where none of the preceding conditions in a series of if or else if statements are met. It acts as a catch-all for unexpected or unhandled cases.

The absence of a default case means that if transferIndex does not match any of the conditions specified in the if and else if blocks, then no code will be executed. This could potentially lead to unexpected behavior or errors if transferIndex takes on an unexpected value.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

 if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98-L107

### Recommended Mitigation
Add the default case 

```diff

if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
+          else
+           {// Add unsatisfactory case  }

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98-L107

##

## [L-4] Use ``safeIncreaseAllowance``\``safeDecreaseAllowance`` for approving tokens instead of unsafe approve() function

It is generally recommended to use the safeIncreaseAllowance() and safeDecreaseAllowance() functions instead of the approve() function when approving tokens. The safeIncreaseAllowance() and safeDecreaseAllowance() functions are defined in the OpenZeppelin SafeERC20 library. This library provides a number of safe functions for interacting with ERC20 tokens, including the safeIncreaseAllowance() and safeDecreaseAllowance() functions.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

17: _token.approve(msg.sender, type(uint256).max);

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L17

##

## [L-5] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards	

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it, except by block-listing the whole protocol.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

148:   token.transferFrom(proxyAddressFrom, msg.sender, amount);
160:   token.transferFrom(msg.sender, proxyAddress, amount);
170:   token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L148




