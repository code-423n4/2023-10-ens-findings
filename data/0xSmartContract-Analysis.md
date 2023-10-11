# 🛠️ Analysis - ENS Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Packages and Dependencies Analysis | Details about the project Packages |
|f) |Other recommendations | What is unique? How are the existing patterns used? |
|g) | Centralization risks 
|h) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-ens#ens-audit-details

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-ens#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Ens](https://docs.ens.domains/) explainer provides a high-level overview of the Project system and the [docs](https://docs.ens.domains/dapp-developer-guide/quickstart) describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-ens#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-ens#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-10-ens#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/176d5743-7a9a-40e9-8bc5-f573399c2dec)

</br>
</br>

## Code Entry Point

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/0b9c6849-b189-4eb4-8e0e-2591ec4edf82)


## delegateMulti()  - _delegateMulti()
Executes the delegation transfer process for multiple source and target delegates;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/d247c669-a506-47ff-9903-ad2cae700fd6)

</br>

## _processDelegation() 
- internal function
Processes the delegation transfer between a source delegate and a target delegate;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/a0bb4513-36c4-4b06-a1c9-8568fcbca1fb)
</br>

## _reimbuse() 
- internal function
Reimburses any remaining source amounts back to the delegator after the delegation transfer process.

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/1ec0ff93-d6de-47f2-b084-2d72861d6131)

</br>

## createProxyDelegatorAndTransfer() 
- internal function
createProxyDelegatorAndTransfer function is an internal function that takes a target address and an amount of tokens as parameters. It is responsible for creating a proxy delegator for the target address (if it doesn't already exist) and transferring a specified amount of tokens to this proxy delegator. The tokens are transferred from the caller of the function (represented by msg.sender) to the proxy delegator's address;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/cd0aa56a-83ee-41bf-8ff4-11b645010d1f)
</br>

## transferBetweenDelegators() 
- internal function
The function retrieves the proxy contract addresses associated with the from and to delegators using the retrieveProxyContractAddress function. This ensures that the tokens are transferred between the proxy contracts, not directly between the delegators;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/bdf9627c-0f2f-496c-91f5-b4222e3cde88)
</br>

## deployProxyDelegatorIfNeeded()
- internal function
The deployProxyDelegatorIfNeeded function is an internal function that checks if a proxy contract for a specific delegate has already been deployed. If not, it deploys a new proxy contract;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/188e2722-86c6-4b12-b031-58763c567fc8)

## retrieveProxyContractAddress()
- private function
retrieveProxyContractAddress function is a private view function that calculates and returns the address of a proxy contract for a given delegate and token. It does not actually deploy a contract but predicts the address where the contract would be deployed;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/ff55ff66-9157-4811-88bd-14ed5612999a)


</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) The tests of the project were designed according to the threat model and it was a part where the team did a great job.


### What could they have done better?
-   1) It is important to have 100% test coverage, the slightest gap in a project's test coverage prevents unexpected major issues from being uncovered.

```js
README.md:
  - Overall line coverage percentage provided by your tests?: Stmts: 100%, Branch: 91.67%, Funcs: 100%, Lines 100%
```
</br>





## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.






##  e) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |                      @openzeppelin/contracts/token/ERC1155/ERC1155.sol , @openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol , @openzeppelin/contracts/utils/Address.sol, @openzeppelin/contracts/utils/math/Math.sol |-  Version `4.3.1` is used by the project, it is recommended to use the newest version `4.9.3`                                                                                                                  |



## f) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ Due to `onlyOwner` powers; It is better to override `renounceOwnership` so that it cannot be used.

##  g) Centralization risks 


- Centralization risk in contracts

There is a risk of centrality in the project as the onlyOwner The owner role has a single point of failure and onlyOwner can use critical functions, posing a centralization issue. There is always a chance for owner keys to be stolen, and in such a case, the attacker can cause serious damage to the project due to important functions.

The code detail of this topic is specified in automatic finding; https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#m01-centralization-risk-for-privileged-functions

Reducing the onlyOwner centrality risk in blockchain projects is essential to enhance security, decentralization, and community trust. Relying too heavily on a single owner or administrator can lead to various issues, such as single points of failure, potential abuse of power, and lack of transparency. Here are some strategies to mitigate the onlyOwner centrality risk:

1-Multi-Signature (Multi-Sig) Wallets: Instead of having a single owner with full control, you can implement multi-signature wallets. Multi-sig wallets require multiple parties to sign off on transactions or changes to the smart contract, making it more decentralized and secure.

2-Timelocks and Governance Delays: Introduce timelocks or governance delays for critical operations or updates. This means that proposed changes need to wait for a specified period before being executed. During this period, the community can review and potentially veto the changes if they identify any issues.

3-Decentralized Governance Mechanisms: Implement decentralized governance mechanisms, such as DAOs (Decentralized Autonomous Organizations) or community voting systems. This allows token holders or stakeholders to have a say in important decisions, making the project more community-driven.

Great summary of GalloDaSballo with examples of centrality risk and how to reduce it in previous Code4rena audits; https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837


## h) New insights and learning from this audit 

1- A broad overview of how the given delegate votes on behalf of the primary delegate and how this authority can be delegated to multiple accounts, and I clearly understood how to code these in smart contracts.

2- Details of Openzeppelin ERC20Votes and ERC1155 code base









### Time spent:
14 hours