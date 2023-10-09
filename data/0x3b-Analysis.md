# Description

ERC20MultiDelegate is a combination of two smart contracts:

- [ERC20ProxyDelegator](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L15-L20) - This is a basic contract used only to store the voting tokens and delegate them to a delegate. This contract is created for every user who uses **ERC20MultiDelegate** to vote or acquire voting tokens.

- [ERC20MultiDelegate](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L25C10-L25C28) - This is the more advanced contract that manages delegations. With it, users can delegate their voting tokens to other users, or, if they have already delegated, change the delegation, or withdraw the voting tokens.

This combination of contracts makes it possible for users to store their delegated tokens in another contract and exchange and trade their representatives, which are ERC1155 tokens.


| Severity | Occurrences |
|----------|-------------|
| High     | 0           |
| Medium   | 0           |
| Low      | 3           |
| NC       | 1           |
| Analysis | 2 hours     |


# Time allocations

|            |            |
|------------|------------|
| Start date | 09.10.2023 |
| End date   | 09.10.2023 |
| **Total**  | **1 day**  |

| Members       | Positions            | Time spent |
|---------------|----------------------|------------|
| **0x3b**      | full time researcher | ~7H        |


# Findings
The codebase was really simple and straightforward. As I expected, there were not many bugs, or at least I didn't encounter anything major. The report is filled with lows and non-critical bugs. Some time was spent exploring some out-of-scope codebases to gather reference points, although there were not many contracts that implemented ERC20MultiDelegate. I was interested in the DAO mechanism used with these voting tokens; however, I was not able to find it. Probably, it is still under development. Approximately 80% of my time was dedicated to attempting to manipulate the rewards, users' vote balances, transfers, and approvals. I wrote multiple tests (~600 lines in Foundry), and the code turned out more secure than I had hoped ): 


# Codebase Quality
The code quality was excellent, featuring simple yet proficient code. However, due to numerous missing pieces of information related to how these vote tokens will be used with other systems, I found myself investing more time than intended in out-of-scope code exploration and extensive documentation reading.

1. **Code Comments**
Comments were excellent in providing more information about the specific logic.

2. **Documentation**
The documentation was well-written, although more information about how these tokens will be used would have been beneficial.

3. **Organization**
The codebase is very mature and well-organized, with clear distinctions between the functions and their purposes.

4. **Testing**
- **Unit Tests:** The unit tests were sophisticated enough to uncover basic bugs. This is sufficient since the codebase was simple and didn't have any major complications or unnecessary complexity. I wrote some tests in Foundry, and the code behaved as expected. It can be noted that more tests are always better, as some tests were too simple, and some scenarios were not tested.


###### ERC20MultiDelegate
This is the main contract in the small system of votes. It is an ERC1155 token contract that implements ERC20Votes internally. It operates on the premise that when users want to delegate their votes to someone else, it creates an **ERC20ProxyDelegator** and sends those tokens in, while the **ERC20ProxyDelegator** delegates them to the delegate. After the tokens are sent and delegated, it mints the delegator ERC1155 tokens with the ID of the delegatee (as `uint256(uint160(delegatee))`). This way, tokens delegated to the same person can be transferred to other addresses while the delegate remains the same.

###### ERC20ProxyDelegator
This is the smaller of the two contracts. It is deployed when a user has received some vote tokens through **ERC20MultiDelegate**. They are generated one per user and store all of the vote tokens, delegating them to the user for whom they were deployed.

# Architecture Recommendations
The contracts provided to us were sufficient to achieve the system's intended goals. They were quite simple, and the code was straightforward, making the audit a pleasant experience.

**Positive Suggestions:**
1. One potential enhancement I recommend is implementing a way for ERC1155 to work with approvals. This would allow users to approve the NFTs and authorize approved users to manage them, such as changing the delegated user, lowering or increasing votes, and so on.

**Some things that may want a second look from the devs:**
1. **Include invariance and fuzz tests** - These are essential for a comprehensive system.


# Centralization risk
The code in its current version is decentralized to the maximum extent. The owner only has the ability to change the URI, which has no effect on the system and is used solely for visuals. The contracts are straightforward and provide users with maximum decentralization, making every action user-dependent. With the current control limit on the owner, even if they are compromised, nothing harmful can happen to the system, as they have no control over the mechanisms.


# Systemic Risks
The current state of the protocol is very secure. I would recommend that the DAO and its proposal generation be examined in conjunction with this code for a full comprehensive audit. For now, the current part of the system seems secure.

Nonetheless, it is crucial to exercise caution when adding new features to the code, especially those involving economic incentives. While they can enhance the protocol's performance and attract users, they also tend to be the most vulnerable points, susceptible to exploitation if not designed and implemented with the utmost care.

**Some potential risks may include:**
1. **Quadratic voting may be less efficient** - This contract makes quadratic voting less efficient since, with less gas, we can achieve a higher spread on the tokens, thereby increasing their efficiency in a quadratic voting system. This makes it more likely that users will attempt Sybil attacks on the DAO and proposals.

# Conclusion
In general, the ENS exhibits an interesting and well-developed architecture. I believe the team has done a good job regarding the code. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
7 hours