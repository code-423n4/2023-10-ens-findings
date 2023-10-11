# Audit Analysis - ENS MultiDelegate
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Flow Diagrams | User flow diagrams explaining possible user interactions |
|d) |Test analysis | Test scope of the project and quality of tests |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-ens#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-ens#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Overview](https://github.com/code-423n4/2023-10-ens#additional-context) Additional Context provides a high-level overview of the Project . 
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-ens#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-ens#scope)|All codes are examined one by one from top to bottom|
|7|Project Audit Reports |N/a |The project is un-audited|
|8|Infographic|[Miro](https://miro.com|I made Miro diagrams to produce Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Attack-Ideas](https://github.com/code-423n4/2023-10-ens#attack-ideas-where-to-look-for-bugs) ,[Main Invariants](https://github.com/code-423n4/2023-10-ens#main-invariants) |This are the main points where the contract can go wrong |


## Analysis of the code base

### ERC20MultiDelegate.sol
`ERC20MultiDelegate.sol ` is the only file in scope . The ERC20MultiDelegate contract is a complex Ethereum smart contract that handles the delegation of ERC20 tokens. The contract allows users to delegate to multiple delegatees to act on their behalf .  It enables token/vote transfers to targets or facilitates reimbursements. Different source/target combinations are permissible, provided the total amounts match up . The contract deploys a proxy delegate for each of the targets, that receives tokens, then grants max allowance to the sender before delegating votes to the specified target address .

### flow diagram :
The flow diagram explains how the delegation logic actually works . 
<img src="https://miro.com/app/board/uXjVNdUvnE8=/?share_link_id=212358283702" alt="Flow diagram "/>


##  Test analysis
Test Coverage is 100% which is best practice . 




##  Documentation 
Documentation was quite poor for the scope of the audit . No external documentation was provided regarding Multidelegation of votes .

## Systemic risks 
 The contract uses uint256[] type arrays to take input of addresses . But an address can be at most `type(uint160).max ` . To ensure proper address generation from uint256 the inputs are casted down to uint160 and then converted to address . THis doesnot ensure the proper validation of the input . If a number is more than type(uint160).max is pushed the txn still sexecutes . this may cause issues . 
##  Centralization risks 
No major centralization risk except the `setUri` funciton . If the `setUri` funciton is used maliciously and URI is changed then all the users will lose their balance and thus track of delegation .


##  New insights and learning from this audit 
The design is quite unique in terms of design . For tracking and managing access control  purpose , it inherits from ERC1155, where `tokenId` in the function `balanceOf(sender, tokenId)` , is the address of the target converted to a uint256. Which I found quite innovative. 
The contract looked pretty well written . 




### Time spent:
Time spent reviewing the code : 18 hours
Time spent writing Analysis report : 3 hours 

### Time spent:
18 hours