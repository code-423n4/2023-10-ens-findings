## Approach taken in evaluating the codebase

The two token standards involved here are ERC20Votes and ERC1155. We first dig into the two tokens' contract source itself, noting down any caveats that one may not be fully aware of. 

The only external non-admin function is `delegateMulti()`, so there is only one logic flow to be working on. After validating that the logic does indeed work, we try to break assumptions by creating transfer scenarios. For example, since the contract itself is an ERC1155, we assess what happens when transferring the token itself, or whether there might be re-entrancy issues related to `ERC1155Holder` implementations.

The code to be auditing is quite short, and it's easy to see the problem we're trying to solve is the limitation of the `delegate` mechanism within ERC20Votes. Perhaps the only guesswork required is what the `ERC1155` token symbolizes, which we summarize in the next section.

## Summary of the codebase

### ENS Token

The ENS token is an ERC20 token with voting capabilities, that is, holders of the token can vote for governance proposals. The voting power is calculated by balance snapshot at a certain block based on the proposal, or a certain formula based on the balance at said time.

By extending OpenZeppelin's `ERC20Votes`, a user can also delegate their voting power to another address to vote on behalf. For example:
- Alice holds $1000$ ENS tokens. This equates to $1000$ units of voting power.
- Alice *delegates* her voting power to Bob.

As a result:
- Alice still has control over $1000$ ENS, but she doesn't hold voting power anymore.
- Bob now has $1000$ voting power, but Bob has no control over Alice's tokens. He cannot spend Alice's tokens, nor transferring it to anyone else.
- If Alice transfers her tokens to anyone else, then Bob's voting power is also transferred accordingly to the token recipient (or the recipient's delegatee, if any).

The downside of `ERC20Votes` is that, delegation is a one-on-one mapping. That is, a user can only delegate *all* of their voting power to *only one* user at a time, or keep all of their voting power. 

The `ERC20MultiDelegate` is created to solve this problem: it allows users to divide their voting power across multiple delegatees (including themselves).

### ERC20MultiDelegate

The `ERC20MultiDelegate` contract is actually an ERC1155 token that is intended to be a wrapper of the ENS token:
- Owning one token with id `target` represents ownership of one ENS token, but with the voting power delegated to `target`. 
- Delegating voting power to `target` equates to wrapping ENS, and minting equal amount of ERC1155 tokens with id `target`.
- Re-delegating voting power from `target1` to `target2` equates to burning tokens of id `target1`, and minting tokens of id `target2`.
- Un-delegating voting power of `target` equates to burning tokens with id `target`, then unwrapping equal amount of ENS tokens.
- Following ERC1155 standards, the `ERC20MultiDelegate` itself is a transferrable token. Transferring a token with id `target` is essentially transferring ENS token itself, but its voting power remains with `target`. The new owner can then re-delegate or un-delegate at will.

The heart of the contract lies in the `delegateMulti()` function, allowing users to seamlessly divide and portion their voting power across multiple voters. This is also the only user entry-point of the contract, aside from standard ERC1155 functions.

## Codebase quality analysis

This codebase is short and easy to understand, given that it only deals with ERC20 and ERC1155, both of which are well-known token standards. Functions are short, well-modularized, and function names are easily understandable. 

We also want to highlight that `_delegateMulti` correctly follows the CEI pattern to prevent reentrancy, as external call happens during minting of ERC1155, which is executed last.

One comment we would have is that there are quite a lot of easy gas-savings techniques can be spotted, and were found by the [automated report](https://gist.github.com/code423n4/33c1c8dad584e9eba48c38c69495fb59). We highlight some of the easily implementable, but impactful, findings in the following section.

## Architecture recommendations

We highlight some of the bot findings that can be easily implemented, but are of high impact, and should pose no security risks:
- **[G-02]**: `token` can be made immutable, as it is never modified after construction.
- **[G-03]** and **[G-11]**: Using prefix increments, as well as `unchecked` blocks, will be very impactful, for gas savings is made *every iteration* of the loop.
- **[G-07]**: In function `getBalanceForDelegate()`, the balance is accessed as `ERC1155(this).balanceOf()`. The external call is not needed, as `balanceOf()` already a public function. 
- **[G-21]**: Functions `Math.max()` and `Math.min()` are called multiple times. Their result can be cached.

Further gas findings can be found in our own gas report submission.

## Centralization risks

There are only two external functions within the contract, `delegateMulti()` and `setURI()`, the latter of which is an admin-only function. The other is independent of any information set by the admin (aside from the token itself, which is immutable).

There are also standard ERC1155 functions that can be entrypoints, however none of them has any centralization risk.

## Systemic risks

While this token wrapper makes it easier to delegate voting power, the fact that delegating being easy may pose a systemic risk by itself: 
- Each proposal has a *voting strategy*, that is, the formula such that the voting power is calculated. There are certain strategies that attempts to limit voting power for those who hold too many tokens. Some examples are:
    - Quadratic strategy: A user's voting power is the *square root* of their balance, instead of their balance.
    - Thresholding strategy: A user's voting power is capped at a certain point.

It is then easier to bypass these strategies by distributing the delegation using the contract, although it is very much possible even without to begin with.

The other systemic risk is that, whenever a new delegate is created, an ERC1155 token is minted. If the "user" is a smart contract, then the `onERC1155Received` hook can be triggered, which constitutes a rather arbitrary external call. Within scope, we have not identified any risk to the token itself, but it may need consideration from a wider perspective.


### Time spent:
12 hours