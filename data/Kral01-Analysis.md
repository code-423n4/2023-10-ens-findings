# ENS-Analysis

## 1. Summary

For this analysis, I will center my attention on the scope of the ongoing audit contest `2023-10-ens`. I begin by outlining the code audit approach employed for the in-scope contracts, followed by sharing my perspective on the architecture, and concluding with observations about the implementation's code.

Please note that unless explicitly stated otherwise, any architectural risks or implementation issues mentioned in this document are not to be considered vulnerabilities or recommendations for altering the architecture or code solely based on this analysis. As an auditor, I acknowledge the importance of thorough evaluation for design decisions in a complex project, taking associated risks into consideration as one single part of an overarching process. It is also important to recognize that the project team may have already assessed these risks and determined the most suitable approach to address or coexist with them. 

## 2. Audit Approach

Time spent: 40 hours

### 2.1 Documentation and scope

The initial step involved examining audit documentation and scope to grasp the audit's concepts and boundaries, and prioritize my efforts. It is to be mentioned that because the scope was limited to a single contract and with a relatively very low SLoC, it was very easy to grasp a general outline of the codebase quickly.

### 2.2 Setup and Test

While the setup was easy, the tests were in hardhat. I was more inclined towards using foundry and a good warden kindly provided us with their foundry [setup](https://twitter.com/0xbtk/status/1711026866344046830) which was very helpful. The already provided tests served as a documentation which provided insight into how the codebase is supposed to be used and will be used when it launches.

### 2.3 Code review

After getting an initial understanding and setting up the tests, I began to thoroughly examine the codebase by entering from the User's perspective and began to explore each process flows to go 'deeper' into the codebase. I commented heavily on the code and taking down notes as I was going through the codebase.

### 2.4 Threat modelling

I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in determining the most effective exploitation strategies. While not a comprehensive threat modeling exercise, it shares numerous similarities with one.

### 2.5 Exploitation and Proof of Concept

After my code analysis, I began to explore each potential vulnerability that came to my mind while writing tests, which fortunately or unfortunately were invalidated by myself. My primary objective in this phase is to challenge critical assumptions, generate new ones in the process, and enhance this process by leveraging coded proofs of concept to expedite the development of successful exploits. I was going through an iterative process of explore, find leads, write tests to test my assumptions and uncover new leads.

### 2.6 Report Issues

The most effective approach to bring more value to sponsors (and, hopefully, to auditors) is to document what can be gained by exploiting each vulnerability. This assessment helps in evaluating whether these exploits can be strategically combined to create a more significant impact on the system's security. In some cases, seemingly minor and moderate issues can compound to form a critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face. 

## 3. Architecture Overview

### 3.1 Architecture summary

Since this is a very small codebase, the logic is very simple. This contract solves the issue of delegation , when using `ERC20Votes` it is all or nothing, ie. the `Delegator` has to either delegate all their voting power to the `delegatee` or delagete no votes at all. This contract solves that by having a proxy contract for each `delegatee`
 that delegates the token to the delegatee and ERC1155 is used to keep track of the Delegator's tokens.

Here is a link to a gist that contains a simple architecture for the protocol

https://gist.github.com/Abelaby/738a53cd54a1ced3e3076b4d8c938202

### 3.2 User Flows

1. Sources and Targets have the same length, in that case we transfer the tokens from one proxy to other, each proxy delgating the tokens to a specific target address. So we burn the nfts of source id and mint the nfts with target id's to the msg.sender.
2. Sources array is passed as empty and we have something in the target, in that case tokens are transferred from the msg.sender to the target proxy address derived, if deployed already than it is used, otherwise for a new target, new proxy is deployed that delegates the tokens. And we mint the nfts to user to the amount sent to target with the id of target address
3. Target array is empty but sources array is not, in this case source is address of the target (yeah confusing first time) to which previously msg.sender delegated. In this case tokens from that proxy are transferred back to msg.sender and we burn the same amount of nft from the user with the id source address.
4. Source array length < target array length — For indexes where in both source and target exist, _processDelegation is called, for rest of the target length indexes createProxyDelegatorAndTransfer is called.
5. Source array length > target array length — For indexes where in both source and target exist, _processDelegation is called, for rest of the source length indexes _reimburse is called.

## 4. Impressions

It was genuinely enjoyable to review this codebase. The code was vey simple in terms of Sloc which helped to quickly grasp the idea and jump into exploring leads.

## 4.1 Comments

As previously mentioned, the code is generally clean and strategically straightforward; even so, comments can further enhance the experience for both auditors and developers. In the context of this codebase, I believe comments are generally effective in delivering value to readers. Comments are crucial because there are two phases of finding bugs in mathematical operations in a code. Usually, the easier one for most auditors is to spot a discrepancy between the intention the comments documenting the code transmit and the code itself. The other one is questioning the mathematical foundations and correctness of the formulas used. Being specific when commenting on the expected outcomes of calculations within the code greatly aids to address the former.

## 4.2 Argument naming is confusing.

In most functions the argument is named as `delegate` while this is true for the most part, the term 'delegate' is used interchangeably throughout the contract. Suppose in case of a `_reimburse`, the call goes to `retrieveProxyContractAddress()` in which the choice `delegate` might not be the best choice.

## 4.3 Use named imports.

Using named imports is a good coding convention that can increase the overall quality of the codebase.

```solidity
import {Address} from "@openzeppelin/utils/Address.sol";

import {Ownable} from "@openzeppelin/access/Ownable.sol";
import {ERC1155} from "@openzeppelin/token/ERC1155/ERC1155.sol";
import {ERC20Votes} from "@openzeppelin/token/ERC20/extensions/ERC20Votes.sol";
import {Math} from "@openzeppelin/utils/math/Math.sol";
```
## 4.4 Internal function name can be prefixed with '_' .

Internal function names should be prefixed with '_' . This naming convention is followed in `_processDelegation` , `_reimburse` , `_delegateMulti` but not in other internal functions used in the contract.

```
function _createProxyDelegatorAndTransfer() internal {}
function _transferBetweenDelegators() internal {}
function _deployProxyDelegatorIfNeeded() internal {}
function _getBalanceForDelegate() internal {}
function _retrieveProxyContractAddress()internal  {}
```

## 5. Conclusion

Auditing this codebase and its architectural choices has been a delightful experience. Inherently simple systems greatly benefit in terms of security due to simplicity, and I believe this project has successfully struck a harmonious balance with its simplicity and carefully executed codebase. I hope that I have been able to offer a valuable overview of the methodology utilized during the audit of the contracts within scope, along with pertinent insights for the project team and any party interested in analyzing this codebase.

### Time spent:
38 hours