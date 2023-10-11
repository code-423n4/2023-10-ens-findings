The ENS multi-delegate contract provides maximum flexibility to the token holder to delegate votes having full control over their ENS tokens by transferring and holding them in proxy delegate contracts. 
In addition, I found the inheritance of the ERC1155 and casting the addresses as `uint256` and using them as token Ids to track balances highly creative.
Some issues that I noticed:

- The lack of any documentation either in natspec or in the developer docs regarding the vote delegation process detailing the various cases for delegating votes. For an adept smart contract engineer it’s possible to reverse engineer the purpose of the functions by reviewing the test file. However for the inexperienced dapp developer or user, this lack of documentation either in code or docs could create severe confusion.
- It is implicitly assumed that the token holders are either EOAs or ERC1155 compatible smart contract wallets. So in cases where the ENS tokens were are dropped to contracts that haven’t implemented the ERC1155 receiver, they won’t be allowed to delegate their votes. If this was intended, then it needs to be clearly documented.
- The lack of events being emitted for important state changing operations such as deploying a proxy delegate for the target or when getting reimbursed by sources.
- Inheriting the ERC1155 contract simply to track the sender’s balances seems wasteful. As it could be achieved by the means of simple mapping variables that can be maintained in storage for the respective senders.
- There are no checks in place to ensure that the sources and targets aren’t the same addresses. Which leads to  unnecessary transactions where tokens are delegated back to the same sources.
- There are no restrictions on the number of sources and targets, so the transfer transaction could potentially run our of gas.

### Time spent:
20 hours