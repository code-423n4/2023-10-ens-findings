# Analysis Report

## Comments for the judge to contextualize my findings

This section contains my submitted issues which include Gas/QA reports and certain issues I was researching but turned out to be invalid. Here are the points I would like to point out:

1. Gas Report - My Gas report includes 8 gas optimizations that save 36839 gas in total. None of these gas findings affect the logic of the code, which is proved by all the tests passing. Each finding shows the deployment and function execution cost separately for the sponsor to select the ones that save the most gas without affecting code readability and maintainability.

2. QA Report - My QA report includs 4 non-critical issues, 1 low-severity issue and 1 recommendation. Out of these, the two most important issues are [N-03] and [L-01]. The [N-03] issue points out code that is unnecessary in the codebase and can be optimized to not only improve code readability but also save some gas. The [L-01] issue points out a missing implementation of token type ID substitution mechanism, which is used by user's to read the onchain pointer (baseUri + tokenId) to their metadata stored offchain on an IPFS solution. The other issues submitted as important as well but not as much as these two mentioned above.

**Unsubmitted issues:**

1. Self-delegation
   Self-delegation is a common issue found in most voting mechanisms. In this codebase, self-delegation is not prevented. Initially, I thought this could classify as an HM issue but there was no risk associated with this and is considered fine (as mentioned by sponsor). Here is a POC that shows self-delegation is possible (providing in case sponsor wants to add it to the test suite)

 - How to add this POC:
   -  Run `forge --version` to see if you have foundry installed
   -  Run `yarn add --dev @nomicfoundation/hardhat-foundry`
   -  Add this `require("@nomicfoundation/hardhat-foundry");` to your hardhat config
   -  Run `npx hardhat init-foundry`
   -  In the test folder, add `testDelegateMulti.t.sol` and paste the below POC
   -  Run `forge test --match-test testUserCanSelfDelegate -vvvvv` to run the test which proves self-delegation is possible

```solidity
File: testDelegateMulti.t.sol
01: // SPDX-License-Identifier: MIT
02: pragma solidity 0.8.19;
03: 
04: import {Test, console} from "forge-std/Test.sol";
05: import {ERC20MultiDelegate} from "../contracts/ERC20MultiDelegate.sol";
06: import {ENSToken} from "../contracts/ENSToken.sol";
07: import {IERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
08: 
09: contract testDelegateMulti is Test {
10: 
11:     ENSToken ens;
12:     ERC20MultiDelegate public emd;
13:     address constant USER1 = address(0x01);
14: 
15:     uint256[] public  _sources;
16:     uint256[] public _targets;
17:     uint256[] public _amounts;
18: 
19:     function setUp() external {
20:         ens = new ENSToken(1e18, 0, 0);
21:         emd = new ERC20MultiDelegate(ens, "");
22:     }
23: 
24:     function testUserCanSelfDelegate() public {
25:         vm.warp(366 days);
26:         //ens.mint(address(this), 10000);
27:         ens.approve(address(emd), 10000);
28: 
29:         uint256 currentAddress = uint256(uint160(address(this)));
30:         _targets.push(currentAddress); //UINT256 version of USER1
31:         _targets.push(currentAddress);
32:         _amounts.push(5000);
33:         _amounts.push(5000);
34: 
35:         emd.delegateMulti(_sources, _targets, _amounts);
36: 
37:         console.log(emd.balanceOf(address(this), currentAddress));
38:         console.log(ens.balanceOf(address(this)));
39:     }
40: 
41:     function onERC1155Received(
42:         address operator,
43:         address from,
44:         uint256 id,
45:         uint256 value,
46:         bytes calldata data
47:     ) external returns (bytes4) {
48:         // Handle the received ERC1155 token, and return the expected function selector (bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))).
49: 
50:         // Add your custom logic here.
51: 
52:         return this.onERC1155Received.selector;
53:     }
54: 
55:     function onERC1155BatchReceived(
56:         address operator,
57:         address from,
58:         uint256[] calldata ids,
59:         uint256[] calldata values,
60:         bytes calldata data
61:     ) external returns (bytes4) {
62:         // Handle the received batch of ERC1155 tokens, and return the expected function selector (bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))).
63: 
64:         // Add your custom logic here.
65: 
66:         return this.onERC1155BatchReceived.selector;
67:     }
68: }
```

## Approach taken in evaluating the codebase

### Time spent on this audit: 3 days

**Day 1**
 - Understand how the codebase works
 - Mind mapping the codebase architecture and function flow
 - QnA with sponsor for doubts
 - Add some initial inline bookmarks for surface-level issues not caught by the bot

**Day 2**
 - Researching possible issue leads in codebase
 - Add inline bookmarks for Gas and QA issues
 - Validate issues bookmarked
 - Create reports

**Day 3**
 - Ask sponsor questions on issues that turned out to be invalid
 - Write Analysis report

## Architecture Recommendations

### What's unique? 

 - ERC1155 for tracking delegations - The ERC1155 standard is usually seen from the perspective of a token. But the sponsor has looked at it from a different perspective of using it to track delegations. This is what makes the code so simplistic in nature and easy to follow.
 - Using ProxyDelegator as middleman - The ProxyDelegator contract acts as a middleman instead of locking the tokens in the ERC20MultiDelegate.sol contract itself. This allows each target to be assigned their own unique ProxyDelegator and allow separation of concerns when multiple people delegate using the ERC20MultiDelegate contract. Another reason why this implementation is unique is because it allows proxy-to-proxy transfers, which allows the delegator to transfer voting power from A to B without losing delegation at any point in time. Additionally, the same point of not losing delegation applies when the delegator wants to transfer ERC1155 tokens to another address owned by him. 

### What's using existing patterns and how this codebase compare to others I'm familiar with

1. ENS VS [Canto](https://github.com/code-423n4/2023-08-verwa/blob/main/src/VotingEscrow.sol)  - Unlike ENS, Canto uses a voting escrow mechanism (inspired by [Curve's veCRV model](https://resources.curve.fi/governance/voting/)). The approach ENS has taken is quite distinct from Canto. The first difference is that ENS uses ERC1155 for tracking delegations to delegates while Canto uses an in-built `locked` mapping that tracks user struct locks which includes the amount deposited in the lock and the delegate voting power is assigned to (defaulted to user - [see here](https://github.com/code-423n4/2023-08-verwa/blob/498a3004d577c8c5d0c71bff99ea3a7907b5ec23/src/VotingEscrow.sol#L268C1-L285C1)). The second difference is that ENS makes use of ProxyDelegators that store/lock tha ERC20Votes token while Canto uses the VotingEscrow itself to store/lock the CANTO tokens sent as msg.value ([see here](https://github.com/code-423n4/2023-08-verwa/blob/498a3004d577c8c5d0c71bff99ea3a7907b5ec23/src/VotingEscrow.sol#L276C9-L281C78)). The third difference is that ENS allows increasing delegate voting power through the [createProxyDelegatorAndTransfer()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L106) function while Canto uses a [increaseAmount() function](https://github.com/code-423n4/2023-08-verwa/blob/498a3004d577c8c5d0c71bff99ea3a7907b5ec23/src/VotingEscrow.sol#L288) which is more complex to understand and requires the user to increase their lock times. Overall, ENS ERC20MultiDelegate.sol is a much more simplistic and flexible contract that takes on a unique approach to delegations and gets rid of the additional complexity and some features Canto provides. 

### What ideas can be incorporated?

 - Adding a time interval field - This time interval field represents the length of a proposal or the wait time before a user can delegate power to someone else. This can help prevent gaining proposal majority by disallowing a single entity from voting from multiple addresses. This interval is just an extra layer of prevention in case an external voting implementation has an error in it that allows this issue to occur. The interval can be set to 0 for external voting systems that do not require a wait-time.
 - Implementing a reward system for honest target delegates - The more the number of users delegate to a target delegate and the longer a user delegates to a target delegate, the greater the rewards for that target delegate for honest behaviour towards its sources.
 - Using ERC1155 tokens as LP tokens in a lending/borrowing system, which can serve as proof for backed ERC20Votes tokens in a ProxyDelegator.

## Codebase quality analysis

The codebase is high quality due to the difficulty to break it and it's simplicity. The specific aspects of why this codebase is rock solid has been included in sections of this Analysis report such as [What's unique?](#whats-unique) and [How this codebase compares to others I'm familiar with](#whats-using-existing-patterns-and-how-this-codebase-compare-to-others-im-familiar-with).

Additionally, the contract has close to 100% test coverage, which adds an additionaly layer of security to ensure all functions work as intended. I would recommend the sponsor to integrate a foundry test suite with fuzz tests as well to ensure the functions work with all types of soure-target inputs.

## Centralization risks

There are no administration roles in this contract, which make it safe to use. The only centralization risk this contract could pose (external to the system) is gaining proposal majority by using the same tokens to vote using different addresses owned by the same entity. This should be mitigated by setting a wait time till the proposal ends before which delegation cannot be reimbursed or transferred to another delegator. This interval/wait-time is just an extra layer of prevention in case an external voting implementation has an error in it that allows this issue to occur. The interval can be set to 0 for external voting systems that do not require a wait-time.

## Resources used to gain deeper context on the codebase

1. Contest README - https://github.com/code-423n4/2023-10-ens/blob/main/README.md
2. Understanding ProxyDelegator's creation code - https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c
3. Another resource for contract creation code - https://www.rareskills.io/post/ethereum-contract-creation-code

## Mechanism Review

### High-level System Overview

![](https://user-images.githubusercontent.com/109625274/274280456-844d42b6-1819-4f0d-a156-2e6c6a876bd0.png)

### Documentation/Mental models

#### Here is the basic inheritance structure of the contracts in scope

![](https://user-images.githubusercontent.com/109625274/274281378-a9b43d38-ad2a-4a2d-a540-0d1e4449a9b7.png)

#### Execution Path for different situations in ERC20MultiDelegate.sol contract

![](https://user-images.githubusercontent.com/109625274/274291802-d91255af-1ac8-4929-b8ff-58c2199af4a8.png)

#### Understanding how array inputs are passed for different situations

The below three instances are the fundamental situations, which can be combined together depending on how inputs are provided.

1. How to get an initial balance to pass [this check](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131) -  Sources are passed as 0 initially, so that [createProxyDelegatorAndTransfer](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L106) is called. This gives the user an ERC1155 balance when processing further delegations and passing the check.
2. How to get reimbursed - For reimbursement, sources are set to the existing target delegates who were previously delegated. This ensures that we enter [this if block](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L101C15-L103C44) and execute _burnBatch for those target delegated.
3. How to transfer from existing delegate to another delegate - For delegate-to-delegate delegation, [_processDelegation](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L100) is executed. This transfers the tokens from the respective proxy delegators following which minting and burning occurs for the source and targets with respect to msg.sender (delegator).

## User-experience weak spots and how they can be mitigated

1. Optimizing gas costs for users - The current implementation of the contract is solid but it does not consider user experience as one it's core aspects. Since this contract is going to be deployed on Ethereum, optimizing gas costs is a must to allow users to transact more often. This does not pose much of a threat as well since the contract logic is quite simple. My gas report includes most gas optimizations that the sponsor should consider since it does not affect the logic of the code (proved by existing working tests).

## Areas of Concern, Attack surfaces, Invariants - QnA

This section includes answers to certain questions/attack surfaces the sponsor wanted addressed. 

1. Check for proper permissions and roles
   - ERC1155 tracks source-target delegations in it's mapping, thus every pair is unique and no clashing occurs. 
   - ProxyDelegators are unique for each target, thus it is not possible for a target delegate to have overlapping ProxyDelegators

2. Ensure that the delegateMulti function handles array inputs correctly
   - Array inputs are handled correctly in all instances i.e. during typecasting uint256 to address and providing input to ERC20MultiDelegate functions and ERC1155's _mintBatch/_burnBatch functions.

3. Validate the logic for transferring between proxy delegators
   - ERC20Votes tokens are transferred correctly
   - ERC1155 tracking is updated correctly through _mintBatch and _burnBatch

### Some questions I asked myself:

1. Are zero address source or delegates possible since the ERC20MultiDelegate contract does not enforce these checks
   - Zero address in sources and delegates are not possible when transferring, minting and burning since 0Z adds address(0) checks and causes reverts if there are any in the arrays.
2. Are retrievals of Proxy contracts and deployment addresses matching?
   - Yes






### Time spent:
15 hours