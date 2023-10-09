# About ENS project: ERC20 multiple delegate contract

`ERC20MultiDelegate` is designed as a system designed by the ENS team to workaround the limitations of normal `ERC20Votes` delegation where you can delegate your votes only to one user (address). Using this contract, users may split delegate their votes to multiple delegatees instead of one.

The need for such a tool/contract is apparent in the ecosystem as other major projects have also implemented their own, in-house, solutions (example the [MaiaDAO ecosystem](https://github.com/Maia-DAO/eco-c4-contest/blob/main/src/erc-20/ERC20MultiVotes.sol)).


# Implementation overview

**_Users choosing to delegate to multiple other addresses will approve the `ERC20MultiDelegate` contract to withdraw the amount of `ERC20Votes` which it wishes to delegate, at first._

Delegation works as follows:
- multi-delegate-contracts are deployed around a single `ERC20Votes` type token, with witch it will work with
- when a user delegates the multi-delegate-contract deploys a proxy for each delegatee
    - once deployed, the proxy gives full token approval over to the delegation contract, it will never change the delegation target and can be reused
- delegating to a target means tokens will be transferred from the delegator to the minimal proxy representing the delegatee
    - by leveraging this mechanism, the ENS team has created a simple and lightweight way of multi delegating without the need to keep a user map delegation address
    - for accounting and withdrawal purposes, delegators receive an `ERC1155` token which represents their equivalent _deposit_, or, to rephrase, the ERC1155 token they receive is indicates to whom they delegated their tokens and is a means to withdraw those tokens
- a user may choose to move his delegated tokens (full or partially) to a different delegator
    - in this case, again if needed, a new proxy will be created if needed
- when a delegator withdraws his equivalent `ERC1155` token will be burned and the multi-contract will move the `ERC20Votes` tokens from the delegatee proxy to himself
- it is important to note, that delegations are `ERC1155` they can be traded as they are, as such, by buying such a token, you actually buy the underlying `ERC20Votes` token

## Roles

In the system we identify 5 roles:
1. contract owner
    - can change the `ERC1155` token URI
2. initial contract deployer (initially is owner but can be changed)
    - decides which `ERC20Votes` will be used
    - at first, can do what the contract owner can do
3. users without previous delegation
    - can only delegate using the system
4. users with previous delegation
    - can delegate using the system
    - if certain conditions are meet can also move delegation
    - can withdraw delegation
5. users being delegate to
    - do not directly interact with this system, they are the beneficiary of any delegation and are mentioned for completion purpose

# Detailed functionality

The project is composed of one contract which encompass the entire delegation logic. 
It has a single entry point [`ERC20MultiDelegate::delegateMulti`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L51-L63) which, depending on the provided inputs, behaves differently. 

```Solidity
    /**
     * @dev Executes the delegation transfer process for multiple source and target delegates.
     * @param sources The list of source delegates.
     * @param targets The list of target delegates.
     * @param amounts The list of amounts to deposit/withdraw.
     */
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```

There are 3 actions this function can take:
- delegate from user to specific address; this also acts as a deposit from the delegator to the proxy representing the delegatee
- withdraw delegation tokens (undelegate); user gets `ERC20Votes` tokens back in accordance to how many `ERC1155` hey burn
- move delegation power from one address to another; important observation is that it can only be done to from delegatees (from whom to take votes) that the user has `ERC1155` tokens for


The following images indicates the ENS Multi-delegate contract execution flow at a method level granularity

![ENS Multi-delegate contract execution flow](https://res.cloudinary.com/duhkspn7a/image/upload/v1696846686/ens-delegate-Overview_r95wov.png)

Details:
- after finishing processing all delegation operations, subsequent burns of `ERC1155` tokens to indicate that a user now dows not delegate to that specific delegatee and minting to indicate to whom it delegates now are done in batch
- `transferIndex < Math.min(sourcesLength, targetsLength)`; condition translates as: if the are both sources and targets provided as inputs then the will do a delegation move, or simply `move-delegated-votes`
    - `_processDelegation` function contains the delegation move logic, as follows:
        - it will first check if the user has enough delegation to move (via `ERC1155` balance checks)
        - create target proxy if needed (since the source proxy already exists)
        - next transfer the tokens from proxies (otherwise no voting power changes would be felt for the delegatee)
- `transferIndex < sourcesLength`; condition translates as: if there are only sources, and no targets, withdraw tokens, or simply `withdraw-votes`
    - `_reimburse` function contains the withdraw logic
        - it simply transfers the tokens back to the `ERC1155` holder and caller
- `transferIndex < targetsLength`; condition translates as: if there are only target inputs, deposit votes and delegate to it
    - `createProxyDelegatorAndTransfer` function contains the deposit logic
        - creates the proxy target if needed
        - transfer tokens from sender to user

## Code quality improvements

1. Although a light contract it lacks detailed documentation, without reading the code, integrators do not have a clear picture of what it does; better document the code. Some other minimal styling are also required but not major issues.

2. There is only one entry point with (`ERC20MultiDelegate::delegateMulti`) with 3 different behaviors, outlined above. The extra complexity added in code to support this does not outweigh the burden of splitting the function into 3 separate functions. It would also be more clear to whoever uses the tool.

# System architecture

The multi-delegation uses `ERC1155` to map delegator to delegatee logic and proxy deployment for token holding and delegation. This presents both advantages and disadvantages:

## Advantages

- a gas efficient implementations since only with new delegatees are proxies deployed, the number of created proxies will slowly decrease over time, when the majority of users interested in a protocol voting have interacted with them
- easy for users to conceptualize, as an `ERC1155` token with the tokenID `0x0...0<address>` directly indicates how much delegation was give, through this system, to an address
- interoperable with pre-existing `ERC20Votes` tokens, no alterations is needed, just for a `ERC20MultiDelegate` to be deployed with the token address
- lightweight

## Disadvantages

- system does not complexly mimic `ERC20Votes` logic. This may cause issues with integrators as to what to expect (especially do to the sever lack of documentation)
    - allows delegation to 0 address (reflected in balance updates, mentioned as an issue separately)
    - when a `ERC1155` transfer occurs, the delegatee of the proxies are still there, wheres when transferring an `ERC20Votes` delegation is undone
- wrapping `ERC20Votes` as `ERC1155` creates an extra economy layer, allowing for MEV/arbitrage opportunities if say, the `ERC1155` token is cheaper then the `ERC20Votes` itself
- proxies are not unique between `ERC1155` collections that have identical underlying tokens; anyone can deploy a new contract, for the same token and have a separate delegations. There is no proxy resurge in this case

# Risks

## Systemic risks

Directly there are no systemic risks to the tool, as it simply facilitates multi-delegation. Indirectly, due to the `ERC1155` economy, an uneducated user-base may bring down the price of the underlying `ERC20Votes` token by selling their `ERC1155` bellow the token price. Direct `ERC20 - ERC1155` price corelation is not something a normal user usually takes into consideration when selling `ERC1155` tokens.

## Centralization risk

As mentioned in the roles descriptions, the contract owner can only change the `ERC1155` metadata URI, which brings the inherited risk of it pointing to a malicious or inappropriate URL. 

It is recommended that this option should be removed. Since anyone can deploy a delegate using any voting token, an ill intended owner can deploy one, users delegate normally and then the owner changes the metadata URI to indicate manipulative content, such as propaganda that: "the governance of X protocol supports the vile act Y, look how many votes it is delegated to it" type of image.


# Codebase audit review

During the audit, there were several issues identified, mostly relating to how a normal voting delegation component should execute. Most issues can be easily fixed or properly mitigated by explicit documenting the behavior.

The auditor and author of this analysis reviewed the code during a 2 day period, cumulating 9.75h of audit plus findings wrap-up work.
The focus was on:
- checking that declared invariants are respected
- delegation behavior would be something that has sense to users
- user roles are correctly separate in code and do cannot impact the protocol in a malicious manner


### Time spent:
9.75 hours