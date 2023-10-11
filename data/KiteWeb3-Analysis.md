# Analysis Report

## 1. Analysis of Codebase 

This analysis report is based on the audit of one smart contracts: `ERC20MultiDelegate.sol`. This contract is part of the ENS project (a decentralized naming service built on top of Ethereum, designed to resolve a wide array of resources including blockchain addresses, decentralized content, and user profile information). 
The Github repo of the project is https://github.com/code-423n4/2023-10-ens

- ` ERC20MultiDelegate.sol`: The contract implements a multi-delegation mechanism for ERC20 tokens that support the ERC20Votes extension. It relies on OpenZeppelin's libraries for standard ERC20 and ERC1155 functionalities and uses Solidity's native features for creating proxy contracts.

## 2. Findings

Most of the findings/vulnerabilities were found by the bots. The list of the bots findings is listed here https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md.

# Additional Findings
## "Approve" return value in the constructor ignored (Medium Risk)
During the audit, it was discovered that the return value of the "approve" function call in the constructor of the ERC20ProxyDelegator contract is ignored. This could potentially lead to unexpected behaviour if the call to "approve" fails.
Recommendation: It is recommended to always check the return value of the "approve" function call. This can be done using a "require" statement.

## "TransferFrom" return value ignored (High Risk)
The audit revealed that the return value of the "transferFrom" function call in the ERC20MultiDelegate contracts is ignored in several places. This is a high risk issue, as the "transferFrom" function call is critical for the correct operation of the contract. If the "transferFrom" function call fails for any reason, the contract may continue executing and lead to unexpected behaviour.
Recommendation: It is recommended to always check the return value of the "transferFrom" function call. This can be done using a "require" statement.

## 3. Architecture Improvements

Consider refactoring the contracts to reduce cyclomatic complexity. The cyclomatic complexity could lead to difficulty in testing and maintenance.

Constructor/initialization function lacks parameter validation.

Adding checks for zero address when setting address state variables.

Eliminating variables shadowing where possible.

Consider to emit Events before external calls (following the best practice of CEI check-effects-interaction)


Improve readability:
-	Adding extensive comments for Assembly blocks 
-	Import declarations should import specific identifiers, rather than the whole file
-	Magic numbers should be replaced with constants
-	Contracts should each be defined in separate files


## 4. Centralization Risk

In the contract, some functions can be called only by having Owner permission (for example ``` function setUri() ```).  This can lead to a centralised risk, to evaluate who is the owner: single EOA, a multisig. 

The Owner can renounce Ownership: the contract has a function to renounce ownership, which can be risky if ownership is renounced unintentionally (Use Ownable2Step instead of Ownable).

## 4. Time Spent
A total of 6 days were spent to cover this audit, broken down into the following:

1st - 2nd Day: Reading and understanding protocol docs, creation of flow and policy management
3rd Day: Focus on linking docs logic to ERC20MultiDelegate and preexisting contracts, coupled with typing reports for vulnerabilities found
4th Day: Focus on different types of strategies contract coupled with typing reports for vulnerabilities found
5th – 6thDay: Sum up the audit by completing the QA report and the Analysis report

## Conclusion
The audit revealed 2 significant findings that should be addressed to ensure the secure and correct operation of the ENS contract. By following the recommendations provided in this report, these issues can be mitigated. Is recommended to conduct thorough testing and potentially another audit after making these changes to ensure no new issues have been introduced.

### Time spent:
48 hours