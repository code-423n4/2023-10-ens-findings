### Introduction
In the following the intention is not primarily to explain how the protocol works, but rather it as an attempt to derive the protocol design from its purpose and investigate any security concerns which thus arise, to thereby confirm that in those regards the implementation is secure.
I have not found any higher risk issues with the contract so this instead serves to demonstrate, as far as possible, the correctness and security of the contract.

### Motivation and general design
An ERC20Votes-token can not only be owned, but also be used to vote. Delegation decouples ownership from the right to vote. Regrettably, ERC20Votes does not support multiple delegations, but one only has the option to delegate one's entire balance. In order to address this limitation `ERC20MultiDelegate` deploys multiple dummy owners (`ERC20ProxyDelegator`) each delegating to a distinct address, and then has them technically (i.e. ERC20Votes-wise) own the delegators' ERC20Votes-tokens. The true ownership (i.e. control) is then accounted for by issuing ERC1155-tokens keeping track of the delegations.

Abstractly speaking, ERC20 maps $\text{owner} \rightarrow \text{balance}$, ERC20Votes maps $\text{owner20} \rightarrow (\text{balance20}, \text{delegate20})$ and ERC1155 maps $(\text{owner1155}, \text{ID}) \rightarrow \text{balance1155}$. We want a map $(\text{delegator}, \text{delegatee}) \rightarrow \text{amount}$. `ERC20MultiDelegate` uses the ID of the ERC1155 map as the key of the ERC20Votes map and we get the map $(\text{owner1155}, \text{owner20}) \rightarrow (\text{balance1155}, \text{balance20}, \text{delegate20})$. It then enforces a bijection between $\text{owner20}$ and $\text{delegate20}$, which means that we obtain the map $(\text{owner1155}, \text{delegate20}) \rightarrow (\text{balance1155}, \text{balance20})$, where we have simply ignored the $\text{owner20}$ output. Finally it enforces equality between $\text{balance1155}$ and $\text{balance20}$ which gives us two equivalent maps, which we can simply write as $(\text{owner1155}, \text{delegate20}) \rightarrow \text{amount}$, which is the desired map, with $\text{owner1155}$ as $\text{delegator}$.

In essence, `ERC20MultiDelegate` is an ERC1155-tokenization of ERC20Votes delegations.

![ERC20MultiDelegate-diagram](https://user-images.githubusercontent.com/112419201/274013275-1e0651f5-d778-418c-88dd-06dcd59fb9a5.png)

### Derivation of implementation
Since ERC20Votes delegates all or nothing, if we want the holder of some tokens to be able to distribute his delegation to multiple delegatees, there **must** be an individual address for each delegatee. This implies that a contract has to be deployed for each delegatee: a _proxy delegator_. Since it is certainly possible that different delegators want to delegate to the same delegatee, it drastically reduces the number of proxy delegators needed if a proxy delegate (delegating to one delegatee) can delegate on behalf of multiple delegators. That is, the proxy delegator has to hold the entire balance of the ERC20Votes-token which is delegated to its designated delegatee, whether this is from one or several delegators. We have now almost fully characterized the proxy delegators. What is missing is how to control transfers to and from these. Since we expect to deploy many of them it seems cheaper to centralize this in one instance, instead of letting each proxy delegator effectively be a token contract by itself. Fortunately, ERC20[Votes] has two kinds of ownership: formal, i.e. balance, and effective control, i.e. allowance. Only the balance is considered by ERC20Votes when delegating. We can therefore let the proxy delegators hold a balance while giving unlimited allowance to a central managing contract for control. We have now fully characterized the actually implemented `ERC20ProxyDelegator`, the specific implementation of which is discussed below.

#### `ERC20ProxyDelegator` - delegation object
The `ERC20ProxyDelegator` contracts are immutable objects, in the sense that they hold no data and that their deployment address depends entirely on the ERC20Votes-token and the delegatee. An instance of an `ERC20ProxyDelegator` therefore constitutes an immutable representation of a delegation. That is, its address acts as a key for the delegatee of the ERC20Votes-token.
They are deployed whenever, and only when, they are needed.
The only variable associated with `ERC20ProxyDelegator` is the amount of the ERC20Votes-token it holds, which is given full approval to `ERC20MultiDelegate`.
The security considerations are therefore that it correctly keys for the delegatee and ERC20Votes-token, which it does by being deterministically deployed using CREATE2 by `ERC20MultiDelegate`, and that only its deployer `ERC20MultiDelegate` has approval to spend its tokens, which is guaranteed by giving full approval to `ERC20MultiDelegate` in its constructor and having no functions of its own to change this.

Each proxy delegator's balance is owed, in various amounts, to one or several delegators, and each delegator may want to delegate to multiple delegatees in various amounts. We thus need a map $(\text{delegator}, \text{proxyDelegator}) \rightarrow \text{amount}$ to keep track of this. This is exactly what ERC1155 provides. The central managing contract, `ERC20MultiDelegate`, is therefore implemented as an extension of ERC1155.

#### `ERC20MultiDelegate` - ERC1155-tokenization of the delegation object
`ERC20MultiDelegate` is an ERC1155-token whose ID is the delegatee, for which an `ERC20ProxyDelegator` is deployed, and the amount of which represents the amount of the ERC20Votes-token held by that `ERC20ProxyDelegator`.
When an amount `a` is first delegated to the address `d`, `a` ERC20Votes-tokens are transferred to an `ERC20ProxyDelegator` representing `d` (let's call this the `d`-proxy), and the delegator is minted `a` ERC20MultiDelegate-tokens of ID `d` (let's call them `d`-tokens) in exchange. These can then be burned/redeemed for the `a` ERC20Votes-tokens held by the `ERC20ProxyDelegator` representing `d`. Alternatively, the owner of the `d`-tokens can move the `a` ERC20Votes-tokens from the `d`-proxy to the `d2`-proxy, in which case his `a` `d`-tokens are burned and he is minted `a` `d2`-tokens instead.
There is thus an equivalence between the `ERC20MultiDelegate`-token balance and the delegations' weights (delegated amounts to delegatees).
Note that ERC1155 also allows transfers of ID-specific balances between holders. In this case this means that one can transfer one's delegation to someone else, who will then own that delegation and right to reclaim the corresponding delegated ERC20Votes-tokens, instead of having to withdraw the delegation and transfer the ERC20Votes-tokens themselves for someone else to redelegate. This has the consequence that if an address obtains `ERC20MultiDelegate`-tokens in this manner instead, it might not be able to reclaim the ERC20Votes-tokens, if it cannot call `delegateMulti()` or cannot receive the ERC20Votes-tokens (e.g. if the ERC20Votes-token implements an `_afterTokenTransfer()` hook checking for token receivability).
Also note that it is possible to transfer the ERC20Votes-tokens to a `ERC20ProxyDelegator` directly by oneself, which would then be delegated accordingly. These are then stuck and forever delegated to its delegatee. Equivalently, one may first convert these to ERC20MultiDelegate-tokens and then transfer those to a dead address. This conflicts slightly with the functionality of ERC20Votes where a delegator can always, at least in principle, retract his delegation.
However, recall that ERC20[Votes] in addition to balance also has an allowance. This is needed for `ERC20MultiDelegate` to transfer tokens on behalf of the delegator. But only transfers of ERC20Votes-tokens from `msg.sender`, without a corresponding burning of ERC20MultiDelegate-tokens, are offered, so the allowance is safely contained by `ERC20MultiDelegate`. (In addition, the `ERC20ProxyDelegator` addresses are hashes of user input (the delegatee), so they cannot be spoofed).

#### Multiple delegation redistribution
In the most general case we want to be able to transform a delegator's distribution of delegations into another distribution, where we may consider the undelegated votes to be delegated to the delegator himself. A redistribution thus takes `s` delegated amounts of total voting power `a` from `s` sources and redistributes it into `t` delegated amounts, again with total voting power `a`, allocated to `t` targets. A simple algorithm is to first transfer all `s` amounts to one location, and then from there distribute it to the `t` targets. This would require `s + t` transfers. A more efficient algorithm, in terms of transfers, would be to let each source sequentially fill up targets one by one. Then for each source, the subset of targets to which it transfers to contains at most one target to which another source also transfers to. There is thus at least one remaining source or target with only one transfer, so the worst case is `s + t - 1`. This is optimized by finding the filling order with the most intermediate perfect fillings where a source's remaining amount exactly fills the remaining amount of a target. The best case is where each source exactly fills one target each, for a total of `max(s, t)` transfers.
Deciding how to partition the transfers can be handled on the front-end side and the manager contract simply needs to process a given set of transfers.
Instead of a single list of transfers, we can implement this as three lists such that `sources[i]` transfers `amounts[i]` to `targets[i]`. (Note that one source or target object may appear multiple times in these lists.) However, while delegators and the proxy delegator contracts are not distinguished with respect to the ERC20Votes delegatability (i.e. having voting power), only delegators should own effective control of the tokens. There is thus a distinction to be made between a transfer between two proxy delegators and a transfer between a delegator and a proxy delegator. `ERC20MultiDelegate` implements this distinction by allowing the `sources` and `targets` lists to be of different lengths and interpret as a transfer between proxy delegates only when there is both a `sources[i]` and a `targets[i]`, and if there is a `sources[i]` but no `targets[i]` then this is interpreted as a transfer from a proxy delegate to the delegator, and if there is a `targets[i]` but no `sources[i]` then this is interpreted as a transfer from the delegator to a proxy delegator. In each case an `amounts[i]` must be assigned. This works because there is no need to transfer both from and to a delegator at the same time since the net total will be either a transfer to or a transfer from the delegator.
The downside of this implementation is the ambiguous meaning of the elements in the `sources` and `targets` lists. The user must ensure that all transfers between proxy delegators are listed in the first indices, followed by the transfers to/from the delegator.

### Invariants
On the one hand we have ERC20MultiDelegate, on the other we have a construct consisting of a ERC20Votes-token with a distributed delegation instead of single delegatee. These two - the implemented protocol and the construct - must be isomorphic.
The invariant of the construct is, except those of ERC20, simply that the sum of the distribution must equal the owner's balance. Let's write the invariant as $D' = d_0' + d_1' + d_2' + ... = B'$, where $D$ is sum of the owner's distributed delegations, $d_i' \geq 0$ is the amount delegated to delegatee $i$ and $B'$ is the owner's balance. We can split the balance into $B' = b_0' + b_1' + b_2' + ...$ such that $d_i = b_i'$.
In the ERC20MultiDelegate protocol, there are two ways of owning tokens. One is as the ERC20Votes-token, the other is as an ERC20MultiDelegateToken. Let's call those balances $V$ and $B$, respectively. An owner's entire balance is not necessarily all delegated within the protocol, so let $D = d_0 + d_1 + d_2 + ...$ be the delegations within the protocol. Thus $B = D$. Total ownership must be constant within the protocol, i.e. $V + B = const$. Taking the difference between before and after calling the contract we get the invariants
1. $d_i, b_i \geq 0$
2. $\Delta V + \Delta B = 0$
3. $\Delta B = \Delta D$

Furthermore, the initial state of the protocol must be that $b_i = 0$, i.e. only ERC20Votes-tokens may exist before.

The amount an owner has delegated to delegatee $i$, $d_i$, is not an explicitly stored value, rather the sum of the delegated amount to this delegatee over all owners is stored. Let's call it $D_i$.

In terms of ERC20MultiDelegate code the variables are defined, for `owner`, as
$V =$ `ERC20Votes.balanceOf(owner)`
$b_i =$ `ERC20MultiDelegate.balanceOf(owner, i)`
$D_i =$ `ERC20Votes.balanceOf(retrieveProxyContractAddress(token, i))`

Since each call of `delegateCall()` only affects a single owner (`msg.sender`) $\Delta D_i = \Delta d_i$, so $\Delta D = \Delta D_0 + \Delta D_1 + \Delta D_2 + ... $.
It is trivial to confirm that $b_i = 0$ on deployment and then $b_i \geq 0$ is ensured by ERC1155.
And $D_i \geq 0$ is ensured by ERC20Votes so if $\Delta B = \Delta D$ holds, since initially $B = 0$, this means that $d_i \geq 0$, which confirms the first invariant.

As can be seen in the below call graph, `_delegateMulti()` calls functions which can be divided into two independent groups based on what state changes they make. `_processDelegation()`, `_reimburse()` and `createProxyDelegatorAndTransfer()` in one, and `_burnBatch()` and `_mintBatch()` in the other.

![_delegateMulti-call graph](https://user-images.githubusercontent.com/112419201/274013287-db4efb8c-e68b-4b5c-addf-f0651f449eae.png)

This means that it doesn't matter in what order we call the two groups. Furthermore, the state changes by `_burnBatch()` and `_mintBatch()` are a subtraction and an addition, which also commutes. Finally, in the for-loop, exactly one function of the first group is called.
All this implies that `_delegateMulti()` with its array arguments can be decomposed into single calls on elements. That is, the argument combinations
`([source], [target], [amount])` - change delegation from single source to single target
`([source], [], [amount])` - delegate from single source to sender
`([], [target], [amount])` - delegate from sender to single target

So we can verify the remaining invariants in terms of the ERC20MultiDelegate code as follows:
```solidity
/*
For any input
(address owner, uint256[] source, uint256[] target, uint256[] amount)
where (source.length, target.length) == (1,1), (1,0), or (0,1)
and amount.length == 1
*/

address proxySource = retrieveProxyContractAddress(token, source);
address proxyTarget = retrieveProxyContractAddress(token, target);

uint256 V0 = token.balanceOf(owner);                        // ERC20._balances
uint256 b0_s = ERC20MultiDelegate.balanceOf(owner, source); // ERC1155._balances
uint256 b0_t = ERC20MultiDelegate.balanceOf(owner, target); // ERC1155._balances
uint256 d0_s = token.balanceOf(proxySource);                // ERC20._balances
uint256 d0_t = token.balanceOf(proxyTarget);                // ERC20._balances

ERC20MultiDelegate.delegateMulti(source, target, amount);

uint256 V1 = token.balanceOf(owner);                        // ERC20._balances
uint256 b1_s = ERC20MultiDelegate.balanceOf(owner, source); // ERC1155._balances
uint256 b1_t = ERC20MultiDelegate.balanceOf(owner, target); // ERC1155._balances
uint256 d1_s = token.balanceOf(proxySource);                // ERC20._balances
uint256 d1_t = token.balanceOf(proxyTarget);                // ERC20._balances

assert(V1 + b1_s + b1_t == V0 + b0_s + b0_t); // dV + dB = 0
assert(b1_s + b1_t + d1_s + d1_t == b0_s + b0_t + d0_s + d0_t); // dB = dD
```
Note that on its deployment `ERC20ProxyDelegator(proxyTarget)` calls `token.delegate(target)`, so `d0_s` is indeed `token.balanceOf(proxyTarget)`, and similarly for `d0_t`, `d1_s` and `d1_t`.

Let's examine each of the argument combinations.

#### `([source], [target], [amount])`
delegateMulti([source], [target], [amount]) executes
```solidity
_processDelegation(source, target, amount);
_burnBatch(msg.sender, source, amount);
_mintBatch(msg.sender, target, amount, "");
```
which can be expanded to
```solidity
uint256 balance = getBalanceForDelegate(source);
assert(amount <= balance);

new ERC20ProxyDelegator{salt: 0}(token, target); // only if not already deployed

address proxyAddressFrom = retrieveProxyContractAddress(token, source);
address proxyAddressTo = retrieveProxyContractAddress(token, target);
token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

_burnBatch(msg.sender, source, amount);
_mintBatch(msg.sender, target, amount, "");
```
We first note that the first two lines can be removed, up to a change only in error message.
The delegate addresses are simply bijectively (in practice, see note below) mapped to `ERC20ProxyDelegator` addresses.
The ERC20ProxyDelegator can be assumed to already be deployed.
Then the only state changes are made by
```solidity
token.transferFrom(proxyAddressFrom, proxyAddressTo, amount); // d1_s = d0_s - amount; d1_t = d0_t + amount;
_burnBatch(msg.sender, source, amount); // b1_s = b0_s - amount;
_mintBatch(msg.sender, target, amount, ""); // b1_t = b0_t + amount;
```
`V1 == V0` so
`V1 + b1_s + b1_t == V0 + b0_s + b0_t` becomes `b0_s + b0_t == b0_s + b0_t`, and
`b1_s + b1_t + d1_s + d1_t == b0_s + b0_t + d0_s + d0_t` becomes `b0_s + b0_t + d0_s + d0_t == b0_s + b0_t + d0_s + d0_t`,
so the invariants hold.

#### `([source], [], [amount])`
delegateMulti([source], [], [amount]) executes
```solidity
_reimburse(source, amount);
_burnBatch(msg.sender, source, amount);
```
which can be expanded to
```solidity
address proxyAddressFrom = retrieveProxyContractAddress(token, source);
token.transferFrom(proxyAddressFrom, msg.sender, amount); // d1_s = d0_s - amount; V1 = V0 + amount
_burnBatch(msg.sender, source, amount); // b1_s = b0_s - amount;
```
`b1_t == b0_t` and `d1_t == d0_t` so
`V1 + b1_s + b1_t == V0 + b0_s + b0_t` becomes `V0 + b0_s == V0 + b0_s`, and
`b1_s + b1_t + d1_s + d1_t == b0_s + b0_t + d0_s + d0_t` becomes `b0_s + d0_s == b0_s + d0_s`,
so the invariants hold.

#### `([], [target], [amount])`
delegateMulti([], [target], [amount]) executes
```solidity
createProxyDelegatorAndTransfer(target, amount);
_mintBatch(msg.sender, target, amount, "");
```
which can be expanded to
```solidity
address proxyAddressTo = deployProxyDelegatorIfNeeded(target);
token.transferFrom(msg.sender, proxyAddressTo, amount); // d1_t = d0_t - amount; V1 = V0 + amount
_mintBatch(msg.sender, target, amount, ""); // b1_t = b0_t + amount;
```
`b1_s == b0_s` and `d1_s == d0_s` so
`V1 + b1_s + b1_t == V0 + b0_s + b0_t` becomes `V0 + b0_t == V0 + b0_t`, and
`b1_s + b1_t + d1_s + d1_t == b0_s + b0_t + d0_s + d0_t` becomes `b0_t + d0_t == b0_t + d0_t`,
so the invariants hold.

### System closure
The above demonstration that the invariants hold assumed calls only to ERC20MultiDelegate from a single owner. We need to be sure that other calls have no effect on the protocol state.

It is obviously possible to change $V$ by transfers of the ERC20Votes-token to or from the holder. This state change is not seen by ERC20MultiDelegate, except for what amount is approved to `ERC20MultiDelegate`. Only the approved amount can be transferred by `ERC20MultiDelegate`. So $V$ should be seen as the amount of the ERC20Votes-token approved by the holder to `ERC20MultiDelegate`, out of his total balance, rather than the ERC20Votes-token balance itself. As per ERC20 only the holder can change this allowance, and it is done independently from ERC20MultiDelegate. This is also safe because calls to `ERC20MultiDelegate` only affect the caller's (`msg.sender`) balance. ERC20MultiDelegate is thus closed with respect to each caller interacting with it.

The `ERC20ProxyDelegator` contracts are similarly related to `ERC20MultiDelegate`. Their entire ERC20Votes-token balance is approved to `ERC20MultiDelegate`, but the amount transferred from them is entirely determined by the ERC20MultiDelegate-tokens. So only the part of their balance that is thus represented by ERC20MultiDelegate-tokens are relevant. Likewise, any amount of the ERC20Votes-token transferred to `ERC20ProxyDelegator` outside of ERC20MultiDelegate is not felt by `ERC20MultiDelegate` or reflected in its tokens, even though whatever is transferred to them will be delegated to their respective delegatee.

The ERC20MultiDelegate-tokens themselves are secured as per ERC1155, and ownership over them is to be interpreted as a delegation. There is thus no issue that they can be transferred as per ERC1155.

As per CREATE2 all `ERC20ProxyDelegator` contracts are specific to the `ERC20MultiDelegate` that deploys them. It is not possible for any other contract to deploy them (to the same address) in its stead.

Thus is it impossible to affect any state on which `ERC20MultiDelegate` relies in a way that is meaningful to ERC20MultiDelegate, except through the intended channel.

#### Remaining state changes
**Votes checkpoint**
It remains to confirm that the remaining state changes, by `ERC20Votes._writeCheckpoints`, behave as intended. This is easy to see, because they happen completely within an ERC20 transfer. Nothing has been altered with respect to how the vote is ultimately delegated; it still happens within the ERC20Votes-token as per ERC20Votes. Therefore the voting power accounting is unaffected.

**`setUri()`**
`setUri()` is the only other function exposed in `ERC20MultiDelegate` (proper), and has so far been neglected in this analysis. It is, however, independent from everything else, modifying only `ERC1155._uri` through an internal function, and is `Ownable.onlyOwner` protected.

### Differences between ERC20MultiDelegate-delegation and ERC20Votes-delegation
An ERC20MultiDelegate-token is equivalent to an ERC20Votes-token, up to a few differences. The main difference, which is the _raison-dêtre_ of ERC20MultiDelegate, is the ability for one owner to delegate to multiple delegatees. But the chosen implementation of this functionality has some other side-effects:

**Independence of ownership and delegation**
The delegation of an ERC20Votes-token must be set by the owner of the token. When the token is transferred a potential delegation does not follow it. In an ERC20MultiDelegate-token ownership and delegation are independent; one can transfer the token without changing its delegation. This is good in that it offers more flexibility. However, it also implies that one can burn ownership of the tokens, while the delegation is intact, permanently locking the delegation. This can be achieved either by transferring ERC20MultiDelegate-tokens to a dead address, or by directly transferring ERC20Votes-tokens to a `ERC20ProxyDelegator`, circumventing the minting of corresponding ERC20MultiDelegate-tokens.
The latter means of achieving this could potentially (albeit unlikely) arise as a user mistake, trying to delegate by direct transfer instead of using the proper pathway through the `ERC20MultiDelegate` contract. To remedy this one could maintain an internal balance indicating how much was sent from `ERC20MultiDelegate` to the `ERC20ProxyDelegator` and allow sweeping of the rest. Either or both of these may be implemented either in `ERC20MultiDelegate` or in the `ERC20ProxyDelegator`.
As for the first of the means, since transferring ERC20MultiDelegate-tokens to a dead address requires that address to be an ERC1155Receiver, this action requires more deliberation. The simple user mistake aspect of it is thus prevented. To completely prevent it one would have to block all transfers of ERC20MultiDelegate-tokens.

### Alternative design
If the protocol wants to support already existing ERC20Votes-tokens the current design based on proxy contracts almost fully precipitates from the ERC20Votes limitation that one address only delegates to a single address. A standalone multi-delegatable token, on the other hand, could be much more efficiently implemented by simply storing the proxies directly as mappings instead of as contracts. Perhaps a success of this protocol will lead to the adoption of such a standard to replace it.

### Notes
**ERC20MultiDelegate-tokens are not theoretically isomorphic to delegations**
Since a ERC20MultiDelegate-token is defined by 320 bits (two address), while it is mapped to 160 bits (one address), each `ERC20ProxyDelegator` is mapped to by $2^{160}$ different ERC20MultiDelegate-tokens. This only means that there are guaranteed to be collisions; a collision is still as unlikely as randomly chosen addresses colliding.
But this of course means that, technically, the protocol is completely invalid.

### Time spent:
30 hours