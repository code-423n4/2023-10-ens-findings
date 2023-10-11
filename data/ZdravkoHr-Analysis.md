# Approach while auditing
During the audit, I have taken the following steps, not necessarily in this order: 

1. Run the tests and make sure nothing fails.
2. Read the code to understand the high-level idea of the smart contract.
3. Read every single test to familiarize myself with how the smart contract is intended to be used.
4. Think of ways to break these intentions or key protocol invariants.
5. Whenever a possibility for a vulnerability arises, test it with hardhat.
6. Engage with other wardens, ask questions when unsure, but also answer the questions of other competitors. This is a really good way to solidify my understanding.

# Architecture recommendations
Currently, version **4.3.1** of [openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) is used. There are some known vulnerabilities with this version. It is recommended to upgrade to a more recent version.

# Codebase quality
The [**ERC20MultiDelegate**](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol)'s code is well-structured, short and concise. It manages to achieve its purpose without additional complexity.

# Centralization risks
There are no significant centralization risks. The only functionality available for only owners is the [**setUri**](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151-L153) function.

# Mechanism review
[**ERC20MultiDelegate**](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) is  built on top of [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes) tokens. A major drawback of ERC20Votes tokens is that each user can either delegate all of his votes to only one address, or delegate nothing at all. It is possible to distribute tokens across different Ethereum accounts and delegate from each account, but that is inconvenient. **ERC20MultiDelegate** takes care of delegating to multiple sources while removing these additional overheads.

The smart contracts tackles this problem by using intermediate contracts, called **proxies**. A new proxy is deployed for each address that has been delegated to. The contract enables users to delegate to multiple targets in a single transaction. When a delegation is made, a proxy is deployed, if not already. The proxy approves the main contract to spend all ERC20Votes tokens on its behalf and also delegates to the specified target. The ERC20Votes tokens of the delegator are then transferred to the proxy. Finally, ERC1155 tokens with the ids of the targets are minted to the delegator. These ERC1155 tokens serve as an attestation that a certain user has rights over an amount of delegations for a given address.

The delegator can move votes from a source to a target any time he wishes to. An amount of ERC1155 tokens get burned for the source and get minted for the new target. The delegator can also transfer back the original ERC20Votes tokens to himself, reducing the delegations of the delegatee. 

The main [entry point](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L116) which make all of this possible is the **delegateMulti** function which calls the internal **_delegateMulti** function. It takes three arguments: **sources**, **targets** and **amounts**.

 - **sources** -> accounts from which an **amount** of delegations will be removed
 - **targets** -> accounts to add an **amount** of delegations
 - **amounts** -> the amount used for **sources** and **targets**

At least 1 **source** or 1 target must be provided. There are 2 possibilities for the arrays' lengths: 

 1. Their lengths are equal -> transfer an amount of delegations from each source to the corresponding target.
 2. Their lengths are different: 
    - `` sources.length > targets.length `` -> **amount** of ERC20Votes tokens are sent from the source proxy to the **msg.sender**, given that an **amount** of ERC1155 tokens are provided to be burned.
     - `` targets.length > sources.length `` -> **amount** of ERC20Votes tokens are sent from the **msg.sender** to the target proxy and an **amount** of ERC1155 tokens are minted to the sender.

# Systematic risks
 - A problem may occur because of an old version of a dependency.
 - When deploying the **ERC20MultiDelegate** contract, if not checked properly, the set token can be malicious. Meaning that it can implement some of the functions that the contract interacts with in a harmful way.

### Time spent:
25 hours