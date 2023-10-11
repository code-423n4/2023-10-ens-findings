# Intro
This analysis is based on ENS ```ERC20MultiDelegate```contract that relates to provision of voting power through ERC20Votes by making it compatible to multi-delegation.

The contract will be used to allow users to delegate voting powers to multiple address using standard ERC20 and ERC1155 functionalities. The feature is based on proxy contract that enables delegating capability in pair to user unique in itself.

# Mechanism
The contracts ERC20ProxyDelegator and ERC20MultiDelegate provide operational working for creating delegation and voting mechanisms using ERC20 tokens.

The tokens will be used in context of allowing delegators to select multiple delegates and process delegation transfers.

The voting power will be given to each delegate according to the amount approved to them using the function ```delegateMulti()```.

# Codebase Analysis
The codebase in scope of reviewing contains two contracts ```ERC20ProxyDelegator``` and ```ERC20MultiDelegate```.

1. **ERC20ProxyDelegator**

- ERC20ProxyDelegator acts as a proxy delegator for voting on behalf of the original delegator.

- Through constructor it approves the sender of the transaction with maximum amount of ERC20Votes tokens and delegates the tokens to a specific delegate.

2. **ERC20MultiDelegate Contract**


ERC20MultiDelegate is a utility contract create a mechanism of allowing delegators to delegate multiple addresses. ERC20MultiDelegate inherits from ERC1155 and Ownable from OpenZeppelin’s library.

The ERC20MultiDelegate workflow depends on multiple function that perform various functions that are explained as below:

- **delegateMulti:**

This function is external which call the internal  ```_delegateMulti``` fucntion to process the delegation transfers. It allows delegators to delegate to multiple source and target simultaneously.

- **_delegateMulti:**

It is internal function with the process of delegation transfers for multiple source and target delegates, by validates the input arrays and process transfer of delegation.

- **_processDelegation:**
The ```_processDelegation``` is an internal function used for processing delegation transfer between a source delegate and a target delegate. processDelegation checks for balance of the source delegate and then transfer tokens.
- **_reimburse:**
The ```_reimburse``` internal function is used for reimbursing any  remaining amount of source back to the delegator after transfer process is compledted.

- **setUri:**

This function is used for storing the metadata of ERC1155 tokens.

- **createProxyDelegatorAndTransfer:**
``` createProxyDelegatorAndTransfer``` act as an internal function to create a proxy delegator contract for target delegate and transferring tokens.

- **transferBetweenDelegators:**

```transferBetweenDelegators``` performs the functionality of transferring tokens between two proxy delegator contracts.

- **deployProxyDelegatorIfNeeded:**

```deployProxyDelegatorIfNeeded``` functionality is related to deploying proxy delegator contract if it’s not deployed already for a delegate.

- **getBalanceForDelegate:**

```getBalanceForDelegate``` as an internal function is used for retrieving token balance for the delegate.

- **retrieveProxyContractAddress:**

- It is an internal function used for retrieve proxy contract address to be used by the delegate for creation code. The code enables delegation and transfer of tokens between delegates.
- ```retrieveProxyContractAddress``` ensures transfers are processed accurately by emiting events to track the delegation process.

# Code Quality

The codebase in certain points ignores standard security practice like access control, checking address validity which are necessary to control potential vulnerabilities like input validation.

- The function ```delegateMulti()``` does not have access control in terms of reentrancy which open potential vulnerability that can be exploited in terms of approving more voting powers or assigning to a unwanted actor.

- The codebase have some complex functionalities like in the ```retrieveProxyContractAddress``` function that use keccak for hash generation.

- The code could benefit from additional Natspec and documentation to enhance its comprehensibility and serve as a reference for other developers. More natspec should be written that will clarify complex logic, especially where complex operations are carried out.

The code base quality in my opinion is not upto the mark as per the standard of ENS protocol and needed to be improved in regards to large number of users ENS protocol currently have.

The quality needs to be considered more in terms of standard secure practice which is necessary to avoid any type of potential exploit that cause loss to the protocol or its user.

# Systemic risk

The risk identified for the code base are summaried as below
-  ```constructor``` in ERC20ProxyDelegator have no access control or input validation for ```_token``` and ```_delegate``` parameters. Lack of input validation is susceptible to security incident like setting address to zero or  if incorrect or malicious `_token` and `_delegate` addresses can be provided.

- ```delegateMulti``` and ```_delegateMulti``` in ```ERC20MultiDelegate``` both functions lacks input validation and limiting number of array passing in parameters i.e., `sources`, `targets`, and `amounts`.
     -  Insufficient checks may lead to unexpected behavior like inadequate handling of amounts could result in numerical issues or DOS due to large number of iteration in for loops.

- ```processDelegation``` use of `assert` is recommend by OZ for just checking internal error and not to be used for validation as it can lead to panic error.

- ```_reimburse``` and ```transferBetweenDelegators``` function lack proper validation as it can be vulnerable to misuse or unexpected behavior if an incorrect inputs are provided by the user.

# Architecture Recommendation
The recommendation for improving the architectural design is made first in area of improving access control by using role-based patterns, improving validiations by using CEIs,  events usage and upgradeability .
- Role-based patterns will allow a users or protocol to have administrative roles and permissions to perform certain action.
-  Enhancing input validation by Check-effect-Interactions CEIs like modifier and require to ensure that inputs must be valid and according to expected conditions.    
    - Implementation of CEIs will be effective and efficient in error handling system to manage unauthorised scenarios and preventing potential vulnerabilities like misuse or unauthorized use to approve or delegate votes.
- The recommendation is also made in area of using events for critical and important transactions.
    - Events are necessary for having a transparent way to communicate with external systems, facilitating better monitoring, analysis, and interaction on-chain.

- The last recommendation is made in area of using upgradeable contract to accommodate future changes and improvements without causing issues on the existing functionality.
    - Upgradeable designs pattern are necessary for moving out of danger if a critical bug is reported which cause danger to assets of protocol and its users.

 # Conclusion
The review of code base provide understanding of improvement that can are required to ensure developing smooth mechanism for delegating voting powers by users to specific addresses through delegation process.

The recommendation provided in the analysis, if implemented will ensure better, secure and efficient mechanism for the protocol with safeguarding their voting powers and any other asset which is necessary for working of the delegation process.


-  **Note to Judge:**
      - I spent around 20 hours reviewing code base that helps me in identifying potential vulnerabilities that existed in the code base and can result in possible exploit.
      - The reviewing helps me in identifying and reporting 01 high, 02 mediums and 02 low issues.
      - The high issue is related to maximum approval of tokens, 1st medium issue is related to reentrancy exploiting and second one is based on not subtracting amount after transfer.
      - All the above findings will facilitate in improving the security as well as working flow of the contract for the purpose of providing delegation platform.






### Time spent:
20 hours