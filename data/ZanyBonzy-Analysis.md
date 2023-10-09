## **Approach taken in evaluating the codebase**

1. Not a lot of documentations/information on the ENS ERC20DelegateMulti contract was provided. We had to rely on previous knowledge of protocols on which its based on - ERC20Votes, ERC1155, and basic governance contracts. A few helpful links from discord mates also helped us to fully understand the contract should work. With the available information, we noted possible risk areas, noted out parts that weren't clear.
2. After understanding how the contract works, we used static analyzers and linters to detect any basic errors, inconsistencies, and typos in the contract after which we started the process of manual code inspection. Here, we carefully inspected each section of the codebase. We focused on areas of interest specified by the sponsor as well as areas we deemed important. We reviewed possible errors arising from imported contracts. We tested out various scenarios, checking out possibilities. We tested owner's permissions, transfer logics, and input data handling. With the use of linters and knowledge of best practices, we assessed the codebase to see if the required programming practices were conformed to. At this stage, we documented all the identified concerns, asked questions, checked if the behaviours we deemed abnormal were intended. We also use solodit.xyz to make comparisons to contracts based on the same protocols, noted the similarities and differences in their implementaions, vulnerabilities found in these protocols and if they exist in the current contract.

## **Architecture recommendations & Codebase quality analysis**
The quality of the ERC20MultiDelegate contract appears to be of moderate quality. Although, the codebase is generally well-structured, has several logical sections and to a point understandable, the lack of documentation and little amount of comments made the contract challenging to audit. We recommend adding these to help users, developers and other future auditors to understand how to use the contract. The `_delegateMulti` function has a high cyclomatic complexity. More events can be added to log important events. This will make it easier to track the state of the contract and troubleshoot any problems. Error handling can also be improved to prevent the contract from failing unexpectedly. Commendably, the test coverage was almost 100% which means a number of bases were covered. However, a number of non critical issues and gas optimizations were uncovered. We recommended using linters and static code analyzers to identify and rectify these issues.

## **Centralization risks**
Centralization risks exist but doesn't seem to be very serious. The onwer's main function is to set token uri. This doesn't look like something that can be really harm the contract users. The use of OZ's ownable contract comes with the slight issue of ownership transfer to addresses that can't handle ownership. For this, ownable2step is recommended. Also important to note that the owner can at anytime renounce ownership and this can leave the contract adminless.

## **Systemic risks**
The use of inheritance and integrations of multiple features can introduce potential smart contract vulnerabilities. Bugs, logic flaws, or improper implementations could lead to exploits. A number of the imported contracts are also outdated. The use of solidity version 0.8.2 means that the system doesn't get the new security and gas optimization updates that come with the newer versions of solidty. There's an unbounded loop on the arrays which can lead to extremely high gas costs for the users. 

## **Conclusion**

The ERC20MultiDelegate contract is used by users to delegate their voting powers to multiple addresses at once. It relies on standard ERC20 and ERC1155 functionalities, creates proxy contracts which is used to enable unique delegation capabilities for the user-delegate pair. We advise following the provided above recommendations and take into account any identified issues.



### Time spent:
46 hours