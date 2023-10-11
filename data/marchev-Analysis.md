# Description

The `ERC20MultiDelegate` is a smart contract designed for multi-delegation of ERC20 tokens supporting `ERC20Votes`. It enables users to delegate voting rights to multiple delegates and perform tasks like granting, transferring, and revoking voting rights in one transaction, resulting in efficient gas usage.

This is achieved by an innovative and interesting approach, combining ERC1155 features for tracking and delegate proxies as a temporary token storage. This clever combination overcomes the limitations of OpenZeppelin's ERC20Votes implementation which works in an all-or-nothing fashion.

The protocol consists of 2 smart contracts:
- `ERC20MultiDelegate` - contains the core logic for delegating, transferring and revoking voting rights as well as tracking delegates as ERC1155 tokens;
- `ERC20ProxyDelegate` - a minimal contract which acts as a temporary token storage enabling delegating `ERC20Votes` tokens to multiple delegates;

**Existing patterns:** The protocol uses standard Solidity and Ethereum patterns. It uses the `ERC20` and `ERC1155` standards, the `ERC20Votes` extension, and `Ownable` pattern for ownership management.

# Approach

The main objective during the analysis was to gather an in-depth understanding of the codebase and its architecture and provide recommendations for improving it.

Given the small codebase size (216 SLoC), the following approach was followed:
- An in-depth line-by-line analysis of the 2 smart contracts - `ERC20MultiDelegate` and `ERC20ProxyDelegate
-  Research of external dependencies, namely the `ERC20Votes` and analysis of any implications of the way it is integrated into the protocol
- Examination of the protocol test suite

# Architecture Description and Diagram

The `ERC20MultiDelegate` is the core contract for multi-delegation.

At its core, `ERC20MultiDelegate` handles delegating, transferring, and revoking vote rights to and from multiple delegates. To fill the gap left by `ERC20Votes` that lacks such functionality, `ERC20MultiDelegate` deploys `ERC20ProxyDelegate` contracts at deterministic addresses using CREATE2 for each delegatee. Voting tokens for delegation are transferred to their corresponding proxy contract, enabling granular delegation. This overcomes `ERC20Votes` limitations, which mandate a token holder to delegate their entire voting power to a single delegate.

`ERC20MultiDelegate` operates as an `ERC1155` multi-token, allowing it to track all delegated voting rights within the contract. Each delegate target represents a unique token with a specific token ID (the `uint256` equivalent of the delegatee address), minted when voting power is delegated and burned when revoked.

It is worth noting that the used approach has certain implications when it comes to using `ERC20Votes` tokens in the context of a dapp using `ERC20MultiDelegate`. Many `ERC20Votes` functions, such as `ERC20Votes#delegates(address)` and `ERC20Votes#delegate(address)`, would no longer align with the concept of multiple delegates.

The following diagram presents a high-level view of the architecture of the protocol:

![ERC20MultiDelegate](https://evc540.pcloud.com/dpZIsdhRuZTcKN4xZ53u7ZXZCTKMykZ3VZZfYVZZ06MDQdlTRb5bBY7Sw7jWNf1PmALX/ENS20MultiDelegate.drawio.png)

At a lower level, the `ERC20MultiDelegate` contract provides a single entry point for interaction - the `delegateMulti()` function.

This function requires three `uint256[]` parameters:
- `sources`
- `targets`
- `amounts`

These parameters function like a multi-directional map. Depending on whether a specific entry has a corresponding matching value or not, the function operates in one of three modes:

- Delegating vote rights (when a `target` lacks a corresponding `source`)
- Transferring vote rights from one delegate to another (when `source` corresponds to a `target`)
- Revoking vote rights (when a `source` has no corresponding `target`)

Now, let's delve into each of these three modes.

## Delegating voting power

When initially delegating vote rights, the `ERC20MultiDelegate` contract deploys a new proxy contract via CREATE2 if it doesn't exist already. This proxy contract represents the delegate `target`. The corresponding amount of `ERC20Votes` tokens is transferred to this proxy contract, which then delegates its full voting power to the delegate `target`.

To keep track and handle internal accounting, an ERC1155 token is minted for each of the delegate `target` and assigned to the `ERC20Votes` owner. This mechanism also holds security implications, as it restricts the transfer or revocation of voting rights solely to the original token holder.

## Transferring voting power

In cases where a `source` corresponds to a `target`, the transfer of the relevant voting tokens takes place. Several checks occur, including verifying if the `msg.sender` has previously delegated adequate voting power to the `source` delegate. It also checks if a proxy contract for the `target` delegate exists or needs deployment. Once all checks pass, `ERC20Votes` tokens are moved from the `source` proxy contract to the `target` proxy contract, effectively delegating voting power to the `target` delegate.

## Revoking voting power

The process here is essentially the reverse of the initial delegation of voting power. All `ERC20Votes` tokens held in the delegatee's proxy contract are returned to their owner, and the associated ERC1155 tokens are burned.

# Codebase quality

In a nutshell, the codebase exhibits excellent quality. It's clear, well-structured, and follows good design practices. The separation of concerns is evident, and functions are concise and aptly named.

Here's a more detailed assessment of various aspects of the code quality:

- **Automated tests:** The codebase has an excellent code coverage. The tests quality is also good and covers critical use cases and scenarios.

- **Code comments:** While the code contains a reasonable amount of comments, some could be more precise and descriptive, particularly in documenting the contracts. Essential functions, like `delegateMulti()`, could benefit from more detailed explanations.

- **Documentation:** Unfortunately, no documentation is provided. Given the innovative approach to delegation to multiple delegates, comprehensive documentation would be highly recommended. This becomes even more crucial for a project of the caliber of ENS.

- **Organization:** The code is well-designed and follows a good structure. There is a clear separation of concerns between the contracts. Functions are short, cohesive, and fulfill their intended purposes.

# Systemic and centralization risks

**1. Liquidity risks:**

The mechanism used by the protocol essentially locks `ERC20Votes` tokens in the vault-like proxy contract which provides positive security implications. However, this creates liquidity risks if these tokens can be used outside of the protocol, e.g. for staking.

**2. Centralization risks:**

The only functionality in the protocol which is restricted to the protocol owner is setting the `ERC20MultiDelegate` ERC1155 URI. Given the lack of documentation, it is hard to say whether this poses any centralization risk  because the semantics of the token URI are unclear.

**3. Concentration risks:**

The `ERC20ProxyDelegator`s contracts accumulate substantial voting tokens and grant unlimited spending permission to the `ERC20MultiDelegate`. If the `ERC20MultiDelegate` is upgradeable, it could pose a risk, allowing potential vulnerabilities to lead to a large voting power takeover.

**4. Loss of voting power risks:**

From a usability standpoint, the protocol could be improved. The `ERC1155` token standard lacks a mechanism to list all owned tokens by an address. Therefore, `ERC20Votes` token holders must remember their delegatees. If they forget, there's a risk of losing voting power. Although recovering this information from emitted events is theoretically possible, it could be impractical due to ERC1155 less robust logs.

# Recommendations

**1. Mechanism to Query Delegates:**

To address the challenge of tracking ownership effectively under the ERC1155 standard, consider implementing a mechanism that allows users to query their delegates. This will help mitigate potential voting power loss.

**2. Evaluate Use of ERC20Votes:**

Assess the necessity of using `ERC20Votes` as the underlying voting token. The multi-delegate approach introduces complexity and indirection, which may not align well with the `ERC20Votes` API. Evaluate the advantages and disadvantages of using `ERC20Votes` compared to a custom implementation or traditional ERC20 tokens.

**3. Comprehensive Documentation:**

 Improve the protocol by providing extensive documentation for both the contracts and the overall protocol. This documentation should include clear, detailed comments and explanations to make the innovative multi-delegation approach easily understandable to developers.
 
**4. Rectify Test Setup Issues and Redability**

Address problems related to incorrect test setups, as exemplified by the 'should be able to delegate to already delegated delegates' test. In this case, the `token` balance of `secondDelegator` is zero. Although technically correct, this test doesn't accurately evaluate the intended scenario due to the balance being zero. It's crucial to ensure that tests precisely assess the expected behavior.

Furthermore, improved test readability can be achieved by employing specific numbers and amounts rather than calculated ones. Using hardcoded values in tests is not only acceptable but also advisable as they serve as fixtures. The use of calculated values introduces the risk of errors and incorrect test setups or scenarios.

**5. Improve Code Comments**

While the code includes some comments, consider enhancing their precision and descriptive quality, especially in contract documentation. Detailed explanations of essential functions would be particularly beneficial to provide a deeper understanding of their workings.

# Conclusion

The ENS `ERC20MultiDelegate` contract demonstrates an innovative approach and well-though design and architecture that addresses shortcomings in `ERC20Votes`. We believe the team has done a good job regarding the code, but some risks have been identified and we suggest that these are addressed.

As per our suggested recommendation we believe that the smart contract can be improved with minor usability enhancements, comprehensive documentation and improved automated tests. 



### Time spent:
11 hours