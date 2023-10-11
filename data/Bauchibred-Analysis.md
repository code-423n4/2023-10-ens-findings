# Analysis

## Approach

The auditing process began with a high-level yet brief overview of the entire ENS ecosystem and how ERC20MultiDelegate.sol would integrate into the broader system. Besides this, numerous hours were dedicated to reviewing findings from previous audits and drawing insights from them (_which unfortunately weren't particularly beneficial given that this codebase is a new one. **However, as they say, no knowledge is lost**_). This was followed by a meticulous, line-by-line security review of the 216 sLOC.

## Architecture Feedback

From what I gather, the ERC20MultiDelegate contract offers a mechanism to facilitate multi-delegation for ERC20 tokens possessing voting capabilities. Its chief objective is to amplify the delegation mechanics of the ERC20Votes standard by facilitating transfers to multiple targets and handling reimbursements.

- **Proxy Delegates**: A unique proxy contract is dynamically deployed for each target delegate. This proxy procures the tokens, provides the sender with maximum allowance, and then forwards the votes to the designated target address. This method ensures effective delegation management while conserving the voting power of the original token holder. To me, this feature stands out as it offers a flexible and scalable multi-delegation solution.

- **Modularity**: The code divides into two main contracts (`ERC20ProxyDelegator` and `ERC20MultiDelegate`), ensuring distinct responsibilities.

- **Inheritance**: Employing OpenZeppelin's standards (`ERC1155`, `Ownable`, and `ERC20Votes`) guarantees industry standard adherence, reducing the chances of basic contract vulnerabilities.

- **Innovative Tracking Mechanism**: The contract's tracking mechanism is notably unique. Inheriting from ERC1155, which typically employs a `tokenId` for individual asset representation, the contract ingeniously utilizes the `tokenId` in the function `balanceOf(sender, tokenId)` to represent the target delegate address converted to a `uint256`. This approach offers a fluid method for managing balances and delegations within the system.

---

## Centralization Risks

- **Ownership**: Using the `Ownable` standard means the `ERC20MultiDelegate` contract has a sole owner with specific privileges, predominantly around setting the metadata URI. It's essential to consider this ownership and ponder if decentralized options or multi-signature ownership might be more apt. Currently, only the URI can be set. However, if users regularly interact with this, positioning a harmful URI might mislead users to erroneous links or scams, which is unintended. Therefore, the admin should either not have the capability to set the URI without a timelock, or the admin role should be designated to a multisig, offering more security layers.

---

## Systemic Risks

- **Dependency on External Libraries**: The contract significantly leans on OpenZeppelin's libraries. While OpenZeppelin holds a commendable reputation, alterations or vulnerabilities in these libraries could introduce risks.

---

## Testability

- The team deserves commendation for the extensive testing provided, given the contract's complexity. The test cases exhibited expansive coverage, spanning 100% across all sections except for the Branch. Well executed.

---

## Other Recommendations

- **Documentation**: Increased comments would elucidate the more intricate code segments. The current documentation doesn't meet top-notch standards, as many security researchers had to inquire about basic questions in the discord chat, questions that could have been avoided with more thorough comments.

- **Gas Efficiency**: Function calls, particularly those nested within loops, require gas optimization. For instance, in the `_delegateMulti()` function, `Math.max(sourcesLength, targetsLength)` is invoked twice, both inside and outside the loop.

- **Enhance Testability**: As previously highlighted, the existing coverage is praiseworthy, but there's always room for improvement. The protocol should strive for comprehensive coverage across all areas, including the Branch section.

- **Implement Better Naming Conventions**: Some function names, like `_reimburse`, might lead to misinterpretations. A more explicit label, such as `_reimburseToSender`, might provide more clarity.

- **Error Handling**: Tailored error messages would augment user experience and facilitate debugging.

- **Integrate More Sanity Checks**: Additional sanity checks, especially concerning transfers between delegates, are crucial. The checks for `to != from` should always be in place, and while the current implementation inherits this check from other contracts, a clear error message isn't relayed. As such, users might be left puzzled about the error source. Therefore, this should be reviewed, and an appropriate message should be provided if issues arise.

## Security Researcher Logistics

The time I invested in this review totaled 15 hours, spread across three days:

- 1 hour dedicated to composing this analysis.
- 2 hours delving into previous ENS contests on Code4rena, accessible [here](https://github.com/search?q=org%3Acode-423n4+-ens-findings&type=code).
- An allocation of 2 hours across the three days was set aside for monitoring discussions in the discord group for the contest.
- The remaining hours were dedicated to multiple perusals of the 216 code lines, identifying issues, and promptly documenting each discovery. Later, some reports were edited for clarity and precision, based on deeper insights into the entire protocol or downgrading them to QA reports.

## Conclusion

The codebase is impressively structured and provided a valuable learning opportunity. Upon deeper reflection, I've reclassified all my findings to QA reports, believing the severity to be less critical than initially assessed. In essence about the added contracts in protocol though, the ERC20MultiDelegate contract presents an innovative approach to address the multi-delegation complexities associated with the ERC20Votes standard.



### Time spent:
15 hours