# ENS Protocol - Analysis 

|Head |Details|
|:----------------|:------|
|Description| A brief introduction to the protocol
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Takeaway from this protocol, recommendations and final review|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Description
ENS is a decentralised naming service built on top of Ethereum, and designed to resolve a wide array of resources including blockchain addresses, decentralised content, and user profile information.

The key contracts of the protocol for this Audit are:
1. ERC20MultiDelegate.sol: The contract implements a multi-delegation mechanism for ERC20 tokens that support the ERC20Votes extension. This allows users to delegate their voting power to multiple addresses in a single transaction. The contract relies on OpenZeppelin's libraries for standard ERC20 and ERC1155 functionalities. It utilizes Solidity's native features for creating proxy contracts, thereby enabling unique delegation capabilities for each user-delegate pair.

## Approach taken in evaluating the codebase
Steps:
- ``Using a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Reading the documentation``: There was no extensive documentation for the code base except some additional context in the readme file of the audit. But, it was enough to get the basic context for the code base for the audit.

- ``Scoping the analysis``: Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external calls, or the code that changes sensitive state.

- ``Manually reviewing the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Unsafe external calls
   - Unexpected state changes
   - Lack of Access Control

- ``Marking vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on. 

- ``Digging deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Performing a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attack vectors
  - Complex state changes

- ``Reporting any problems found``:  If you find any problems with the code, you should report them to the developers of ENS protocol. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis
There is only 1 contract in scope for this Audit with total nSloc of 216 and 5 external imports. Hence, the code base is pretty small. Is was nice to see almost 100% test coverage for the code. The code has necessary inline documentation required for this audit. The code base is pretty easy to understand and comprehend and is written very well. This codebase uses ERC20 Votes which is an extension of ERC20 that allows users to utilize their tokens as voting power. The users are  also minted ERC1155 tokens that track his ownership of the ERC20 Vote Tokens. This code base also adheres to some of the  the best practices like using name imports, indexed event parameters, view and pure functions, etc. 

## Architecture recommendations
This code base is pretty simple architecture wise as there are not much components to interact for the end user. In fact external users can only interact with ``delegateMulti()`` function. In this function,  different source/target combinations are permissible, provided the total amounts match up. The contract deploys a proxy delegate for each of the targets, that receives tokens, then grants max allowance to the sender before delegating votes to the specified target address. For tracking, it inherits from ERC1155, where `tokenId` in the function `balanceOf(sender, tokenId)` , is the address of the target converted to a uint256 which I found quite innovative.

Even though, the architecture is quite simple, using the ``delegateMulti()`` function is not as straight forward. On first call, you need to provide empty ``sources[]`` array. This ensures all the targets are deployed on first call. After that, if you want to delegate from sources to target, you need to provide equal length of sources[] and target[]. If length of sources[] > target[], any remaining source amount is transferred back to ``msg.sender`` and if length of target[] > sources[], new targets are deployed.

To reduce the complexity, these above mentioned features can be separated into different functions. Even though the logic used in the ``delegateMulti()`` perfectly handles every case given the correct input, but it would be easier for external users to interact with this code base given multiple functions are available. 

## Centralization risks
There are no centralization risks in the code base.

## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion
ENS employs the ERC20Votes and ERC1155 standards to manage delegation and token metadata, respectively. This codebase allows users to delegate their voting power to multiple addresses in a single transaction. The contract relies on OpenZeppelin's libraries for standard ERC20 and ERC1155 functionalities. The codebase is simple and very secure.

## Time-spent 

15 Hours 

### Time spent:
15 hours