## **Approach taken in evaluating the codebase**

The ENS ERC20DelegateMulti contract was not well-documented, so we had to rely on our knowledge of similar protocols and information from Discord to understand how it works. With the available information, we noted possible risk areas, noted out parts that weren't clear.

After understanding how the contract works, we used static analyzers and linters to find basic errors, minor issues and typos. Then, we manually inspected the codebase, focusing on areas of interest specified by the sponsor and areas we deemed important. We also reviewed possible errors from imported contracts, tested various scenarios, and checked owner permissions, transfer logic, and input data handling. We used linters and our knowledge of best practices to assess the codebase and make sure it followed required programming practices. We documented all identified concerns, asked questions, and checked if any abnormal behavior was intended. 

We also used solodit.xyz to compare the contract to other contracts based on the same protocols, noting the similarities and differences in their implementations, as well as any vulnerabilities found in these protocols that might exist in the current contract.

## **Architecture recommendations & Codebase quality analysis**
The ERC20MultiDelegate contract is generally well-structured and understandable, but the lack of documentation and comments made it difficult to audit. We recommend adding these to help users, developers, and future auditors understand how to use the contract. 

The _delegateMulti function has a high cyclomatic complexity, meaning it is difficult to understand and test. We recommend refactoring this function to make it more modular and easier to understand. 
Events should be added for when the delegator is or reimbursed, when the target gets transferred tokens directly from the delegator. Error handling can also be improved to make it easier to track the state of the contract and troubleshoot any problems.

Commendably, the test coverage was almost 100%, which means a number of bases were covered. However, a number of issues were uncovered. We following the recommendations provided by auditors to rectifies these issues.

## **Centralization risks**
The contract has some centralization risks. The owner's main function is to set the token URI. The contract uses OZ's Ownable contract, which can be risky if the ownership is transferred to an address that can't handle it. It's recommended to use Ownable2Step instead. Also, the owner can also give up ownership of the contract, which would leave it without an admin. 

## **Systemic risks**
The use of inheritance and external contracts can introduce vulnerabilities into smart contracts. This is because bugs, logic flaws, or improper implementations in inherited or external contracts can be exploited. Additionally, outdated contracts, OZ 4.3.1 and Solidity versions 0.8.2 contain known vulnerabilities which can be exploited. Updating to later versions of Solidity can provide the with new security and gas optimization updates. 

## **Conclusion**

The ERC20MultiDelegate contract allows users to delegate their voting power to multiple delegates at once. It does this by creating proxy contracts for each user-delegate pair. It relies on standard ERC20 and ERC1155 functionalities and a host of other inheritances.
To ensure that the contract stays secure and works well, we advise following the recommendations that have been provided, regularly upgrading the contract and its dependencies and conducting constant audits.




### Time spent:
40 hours