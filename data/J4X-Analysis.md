## Introduction
This analysis focuses on the ENS audit hosted on Code4rena, which pertains to the ENS (Ethereum Name Service) smart contract. The audit process aimed to identify potential vulnerabilities, propose mitigation strategies, and optimize gas usage within the codebase.
```
| Spec        | Value       |
| ----------- | ----------- |
| Name        | ENS         |
| Duration    | 6 days      |
| nSLOC       | 216         |
```
## Methodology
The audit process was divided into three distinct phases:
### Recon
In the Recon phase, I conducted a preliminary review of the codebase to gain a broad understanding of its structure and functionality. This initial pass allowed me to familiarize myself with the code before diving deeper. Additionally, I examined the provided documentation and gathered supplementary information from online sources and the audit's Discord channel.
### Analysis
The Analysis phase constituted the primary focus of the audit, where I conducted an in-depth examination of the codebase. My approach involved meticulously documenting suspicious code sections, identifying minor bugs, and envisioning potential attack vectors. I explored how various attack scenarios could manifest within the code and delved into the relevant functions. Throughout this phase, I maintained communication with the protocol team to verify findings and clarify any ambiguities.
### Reporting
After compiling initial notes during the Analysis phase, I refined and expanded upon these findings. I also developed test cases to complement the identified issues and integrated them into the final report. Furthermore, I generated Quality Assurance (QA) and Gas reports, alongside this comprehensive analysis.

## System Overview
The system revolves around an `ERC1155` implementation that enables users to delegate their voting power to multiple addresses simultaneously. This is in contrast to the standard `ERC20Votes` implementation where users can only delegate tokens to a single delegate. To facilitate this functionality, the contract offers three key functionalities:

### 1. Delegation
In this scenario, a user delegates voting power to a specific address. To achieve this, the user provides a target address without specifying a source address. The process includes:

- Checking if the target proxy already exists; if not, it's deployed.
- Transferring the specified amount of tokens from the caller to the proxy.
- Minting `ERC1155` tokens from the user to the target.

### 2. Transfer Between Delegates
In this scenario, a user transfers their delegation power from a source to a target. The steps include:

- Checking if the source has a sufficient balance to transfer.
- Deploying a proxy for the target if it doesn't already exist.
- Transferring tokens from the source to the target.
- Burning source `ERC1155` tokens from the user.
- Minting target `ERC1155` tokens to the user.

### 3. Reimbursement

Here, a user seeks to transfer tokens out from a delegate, effectively decreasing or removing their delegated voting power. The user specifies a source without a target, and the process involves:

- Transferring the tokens from the proxy to the user.
- Burning source`ERC1155` tokens from the user.

The contract also supports multiple transactions in a single call, using arrays. However, it's worth noting that delegation and reimbursement functionalities cannot be combined in the same transaction. The decision on which function to execute relies on the length of the arrays rather than the presence of addresses.

Recommendations:
As stated by the project team it is intended that the contract can also be used with other tokens that implement `ERC20Votes`. Unfortunately the current implementation leads to problems with multiple special implementation of `ERC20` like FOT tokens, rebasing tokens, tokens that transfer the balance if `type(uint256).max` is passed as well as tokens that don't revert on transfer. As according to the project team it was mentioned that FOT(fee-on-transfer) and rebasing tokens will not be supported this needs to be mentioned in the documentation. Additionally there either need to be fixes implemented for the other cases or these tokens also need to be excluded in the documentation.

Additionally the functionality of no Delegation and Reimbursement being possible in the same transaction due to the decision on which function is being chosen relying on the length of the arrays (not the addresses being zero) requires a user, if he wants to change multiple delegations at once, to precompute the most efficient path to only need to call the function once. If this intended the functionality can stay as it is, if it would be intended that both can be called in one transaction the functionality inside the loop will need to be restructured.
## Potential attack vectors
Given the concise and well-structured codebase, the potential attack vectors are limited. Nevertheless, during the research, a few possible attack vectors were identified:

- Malicious users could delegate more tokens than they possess.
- Malicious users could delegate the same tokens multiple times.
- Malicious users could steal tokens delegated by others.
- Malicious users could freeze tokens delegated by others.
- Malicious users could retrieve tokens while delegations are still active.

## Testing
The test suite includes multiple test cases for the various functionalities of the `ERC20MultiDelegate` contract. These tests effectively cover all three contract functionalities. However, there are several test scenarios missing from the current implementation.

- testcase 'should be able to delegate multiple delegates in behalf of user' does not check if the deployer has all the tokens back at the end
- missing test for no target/source but an amount provided
- 'should be able to re-delegate to already delegated delegates' incorrectly includes variable names starting with "revert" which is confusing
- No tests for the `ERC1155` functions (especially `_safeTransferFrom()`)

Recommendations:
- Add the missing test scenarios mentioned above to the testsuite
- Fix the mentioned issues in the testsuite
- Also add additional fuzz testing (could be done using foundry)

## Systemic Risk
The non-upgradeable nature of the entire protocol and the fact that the only way to retrieve users' delegated tokens from the proxy is through the contract necessitate a robust focus on the security of the `ERC20MultiDelegate`. This is crucial to prevent user tokens from being frozen indefinitely.

## Code Quality
The code overall is nicely structured and kept very short and simple. As no additional documentation is provided, developers need to understand the functionality directly from the code. Unfortunately the code is missing NatSpec documentation at certain parts and would also profit from more inline comments.

Recommendations:
- Add full NatSpec documentation to every function and event
- Write a simple documentation file that also shows the 3 different functionalities
- Add more inline comments to make the code easier to understand.

## Conclusions
The audit spanned a full six days and comprised three phases:

- Day 1: Documentation review, initial code inspection, and gaining an understanding of the system's functionalities.
- Days 2-5: In-depth code analysis, vulnerability identification, and bug hunting.
- Day 6: Reporting findings, including bug reports and this comprehensive analysis.

Overall the codebase is very well constructed and most of the recommended patterns were followed. A key issue is the current inconcise definition of which `ERC20Votes` tokens are supported as well as missing documentation. Additionally if the `ERC1155` tokens should be tradeable later on, the downcasting issue needs to be fixed too.

Total time spent: 32 hours.


### Time spent:
32 hours