
### Ambiguous meaning of `sources` and `targets`
If `sources.length < targets.length` and `i < sources.length <= j < targets.length` the meaning of `targets[i]` is not the same as `targets[j]`. `targets[i]` receives `amounts[i]` **from `sources[i]`**, whereas `targets[j]` receives `amounts[j]` **from `msg.sender`**. `targets[i]` implies a redelegation; `targets[j]` implies an initial delegation.
Similarly, if `targets.length < sources.length` and `i < targets.length <= j < sources.length`, then `sources[i]` sends `amounts[i]` **to `targets[i]`**, whereas `sources[j]` sends `amounts[j]` **to `msg.sender`**. `sources[i]` implies a redelegation; `sources[j]` implies a withdrawn delegation.
Thus the index ordering of delegations supplied to `delegateMulti()` is critical up to being in the first or last part.

For example, suppose we intend for sources `s1`and `s2` to send to target `t` and source `r` to send to `msg.sender`, then we can supply `([s1,s2,r],[t1,t1],[a1,a2,a3])`, which is equivalent to `([s2,s1,r],[t1,t1],[a2,a1,a3])` but not to `([r,s2,s1],[t1,t1],[a3,a2,a1])`.
It seems an easy user mistake to confuse the order in having to put redelegations first, and withdrawn and new delegatees last.

Consider the possibility of making this distinction explicit by having `sources`, `targets` and `amounts` be of equal length (thus explicitly only redelegations) and adding three new parameters `undelegations` and `newDelegatees`, one of which must be empty, and `userAmounts`, of the same length as the non-empty one.

### No events emitted for new delegations and withdrawal of delegation
Only `_processDelegation()` emits an event, `DelegationProcessed`. Consider whether not `_reimburse()` and `createProxyDelegatorAndTransfer()` should not also emit events. If so, `DelegationProcessed` can be emitted at the end of `_delegateMulti()`, where then `address(0)` might be emitted as the `source` or `target`. This makes sense given that is how `msg.sender` is represented in the parameters of `_delegateMulti()` (i.e. by not being present at all).

### Votes can become permanently delegated
ERC20Votes always allows the holder to change his delegation. Conversely, a delegation cannot be permanent. By deploying inert `ERC20ProxyDelegator` contracts with a permanent delegatee ERC20Votes-tokens may be sent to these which are then stuck and permanently delegated to the delegatee. Equivalently, one may first convert the ERC20Votes-tokens to ERC20MultiDelegate-tokens and then transfer those to a dead (but an ERC1155Receiver) address.
Thus, if ERC20MultiDelegate is to be considered an extension of the ERC20Votes standard, it breaks this aspect.
Transferring directly to the `ERC20ProxyDelegator` could potentially happen as a confused user mistake. To remedy this one could maintain an internal balance indicating how much was sent from `ERC20MultiDelegate` to the `ERC20ProxyDelegator` and allow sweeping of the rest. Either or both of these may be implemented either in `ERC20MultiDelegate` or in the `ERC20ProxyDelegator`.
Since transferring ERC20MultiDelegate-tokens to a dead address requires that address to be a ERC1155Receiver, this action requires more deliberation. The simple user mistake aspect of it is thus prevented. To completely prevent it one would have to block all transfers of ERC20MultiDelegate-tokens.

### Unused `Address`
Lines 4 and 26 may be removed as the imported `Address` is never actually used.

### ERC20MultiDelegate cannot be deployed with updated version of Ownable
`ERC20MultiDelegate` is OpenZeppelin `Ownable` but lacks the parameter `address initialOwner` which [has been added to the new version of Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/793d92a3331538d126033cbacb1ee5b8a7d95adc/contracts/access/Ownable.sol#L38).