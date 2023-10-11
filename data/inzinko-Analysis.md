Analysis of the protocol

The Mulitdelegate functionality that was audited in this audit, involves one of the most simple yet interesting most intelligent ways of utilizing the ERC20Votes Functionality of the ERC20, Then combining it with the Multi tokens functionality of the ERC1155 Contract, This allows a User who wants to leave out complex or believes that their Tokens will be better in the hand of others who have more knowledge about the protocol being voted upon, this will not only help the delegator but also the protocol has more informed participants will have a say in the most important decisions in the ecosystem. Before we dive into the main analysis, during this audit I made it a priority to explore the first functionality of Governance that the ENS protocol uses in making decisions, I found the Tally a service that allows decentralized voting by Protocol community members, on a proposal created by the protocol and make decisions based on what the community members decide on. Users can assign their voting rights, initiate or approve proposals for the allocation of DAO resources, oversee a protocol, and implement smart contract updates—all directly on the blockchain. This service has been utilized by the ENS protocol to Create 18 proposals and a total of 11.93k voters have participated in the voting process. This service or The Governance Contract also allows the ability to delegate or undelegate votes. To partake in governance votes, it's essential to establish delegation. This can be either by authorizing others or by casting delegator votes. Should the delegator prefer to personally weigh in on proposals, assign voting strength to the delegator's address. Alternatively, the delegator can empower another community member's voting rights and allow them to represent the delegator's stance.

To Understand how everything works, I have to guess the ENS management is looking for a way to also Multdelagte tokens to different users in the protocol, just like what is currently being used in tally. 

1. A Proposal is created by the members of the protocol, about some things that need to be changed in the community or just hearing people's opinion about a particular issue, the protocol may implement some kind of threshold to be able to create a proposal, since that is out of the scope of this audit, we won't go further than this

2. Individuals can vote or delegate their tokens which represents their votes to other they trust to make informed decisions or share the same view as them.

3. On the currently used Delegate functions used on tally, the users delegate their votes and undelegate them to themselves, so this essentially means that to participate in the voting a user has to delegate their votes to themselves or delegate them to trusted individuals, and when they feel like not delegating anymore they have to undelegate and delegate those votes to themselves

4. What ENS is trying to achieve with the new functionality to be added to their DAO, is the ability to delegate to multiple individuals, a user can call the _delegateMulti function and delegate their votes to multiple targets, it is that simple the _delegateMulti function makes it so that by delegating your Votes Tokens, you are being minted an equivalent amount of tokens of the actual target you are delegating to, this is made possible by a clearly intelligent method of using the ERC1155 Contracts as a way of effectively bypassing the effect or requirements of the ERC20Votes, which prevents users from delegating their vote to more than one person, by utilizing this crafty method, the ENS Developers have overcome two issues the ERC20Votes issue and also the Safety issue, in which the delegator can remove their delegations at any time they want as they are the rightful owner of the tokens sent to the Proxy contract. This makes The multi delegate functionally seamless as the Delegators easily can delegate and undelegate at any time they actually wish to.

The _delegateMulti Function 

All the Main functionality of the ERC20MultiDelegate contract lies in the _delegateMulti function, a user want to delegate they call the function and pass in these parameter 

Source [], is an array of sources which the user wants to send the vote tokens from, these sources are ERC20Proxy Contracts that keep track of both the delegate and the Vote Tokens, a number of proxy contracts can be entered into this array and the Vote Tokens here will be sent to the destinations, a User can also not Included any address in the array, that indicates or will prompt the user to be the one responsible for the source of the Vote Tokens

Targets [], These are the addresses of the proxy contracts the Votes tokens are to be sent to This also is an ERC20Proxy contract,

Amounts [], an array of amount of Vote Tokens to be sent to the intended target proxy contracts 

The function then loops through the arrays and performs some conditional specific actions, let's explore some of these conditional actions which the function will perform 

IF THEY ARE BOTH SOURCES AND TARGETS TO PERFORM AN OPERATION

If this conditional statement is true, then the _processDelegation function is called, this function Processes the delegation transfer between a source delegate and a target delegate. It checks for the balance of the source that would be backing the amount to be sent to the target. Deploys a proxy contract if the target does not have one yet then proceed to transfer the Vote Tokens to the Proxy Contract of the target.

IF THEY ARE REMAINING SOURCES AMOUNTS IN THE ARRAY YET TO BE UTILIZED

This happens when after the delegations there are any more remaining source amounts, then the function decides to call the _reimburse function to send the amounts back to the delegator from the proxy address.

IF ANY TARGETS ARE LEFT 

This calls the createProxyDelegatorAndTransfer function to create a proxy contract for the target and transfer the amount specified. Allowing the msg.sender to transfer their Vote tokens to a delegate.

The Trick that made the whole functionality that makes the whole functionality all an art, is the _mintBatch and _burnBatch at the end of the _delegatorMulti function, this two methods react based on actions performed by the user in the _multiDelagte functions

If The user Performs a Transfer to a proxy contract, the _mintBatch function mints tokens for the user that represents the amount of tokens delegated to the targets, which is actually a token of the delegate address represented in a uint256, then updates the balances in the ERC1155 contract

If the user performs a Transfer from a Proxy contract to another proxy contract, the _burnBatch function is called to burn any tokens that indicate that they have tokens that are delegated to the user, and the calls _mintBatch that will mint the new destination target tokens for the user.

All this makes the ability to delegate a user Vote Tokens to multiple delegates seamlessly and effectively..

Undelgation 

The Undelegation implemented on the Tally platform to remove Votes Tokens from the delegates back to the user is also beautifully implemented, in the _delegateMulti function, by not entering targets to the function the user is implying that they want to be reimbursed with the amount and those amounts will be transferred from the proxy contract back to the user, and their tokens are effectively burnt completing the Undelgate Process.

FINDINGS AND FUTURE IMPLEMENTATIONS CONSIDERATION

The Governance Functionality has a solid codebase with a much smarter way of bypassing the ERC20Votes restrictions, Some of the Findings found were

The Amount not being checked might lead to united consequences for the User who calls the _delegateMulti function and has an undesired result.

A user using a smart contract as a means to act as a delegator might have problems when trying to delegate their Vote Tokens.

Future Considerations 

After the audit process is completed, and the functionality is being added to the main DAO protocol the ENS Team should consider the possibility of attacks that won't be able to be determined from this scope of the protocol. Attacks like Flash Loans are to be considered as critical damage to the integrity of the Governance. These mitigations are already likely considered and added to the Delegate functionality in use already, which is why the same mitigation must be considered and implemented into this Functionality, to prevent malicious users from manipulating the Governance system. 


### Time spent:
51 hours