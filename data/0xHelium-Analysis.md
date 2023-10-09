# ERC20MultiDelegate Contract Audit Report

## Overview

The ERC20MultiDelegate contract is a utility contract designed to manage delegation of voting power for ERC20 tokens that support the ERC20Votes extension. This contract enables users to delegate their voting power to multiple addresses in a single transaction. It relies on OpenZeppelin's libraries for standard ERC20 and ERC1155 functionalities.

This audit report provides an analysis of the ERC20MultiDelegate contract, including its functionality, security considerations, and potential vulnerabilities.

## Audit Timeline

### Day 1 - System Familiarization:

- Gain a comprehensive understanding of the ENS protocol, its architecture, and its overall functionality.
- Review documentation and whitepapers to establish a baseline knowledge.
- Begin preliminary codebase exploration to identify key contract components.

### Day 2 - Contract Analysis:

- Deep dive into the core contracts and smart contract interactions within the ENS multidelegate protocol.
- Identify and document critical contract functions, state variables, and external dependencies.
- Focus on Multidelegete, ProxyDelegator, their roles, and security measures.

### Day 3 - Security Assessment:

- Identify potential attack vectors, edge cases, and vulnerabilities in the Centrifuge protocol.
- Thoroughly analyze the delegating mechanisms, transfer between proxy, and reimbursement processes.
- Investigate the handling of multiple currencies and associated conversions.
- Review the access setup and governance mechanisms for potential weaknesses.
- Create Proof of Concepts (PoCs) to demonstrate any identified vulnerabilities.
- Assess the protocol's resistance to common attack scenarios.

### Day 4 - Report and Recommendations:

- Compile findings and vulnerabilities discovered during the audit.
- Prioritize and categorize vulnerabilities based on their severity.
- Provide detailed recommendations for mitigating identified risks.
- Summarize the audit results, highlighting key areas of concern.

## Codebase Quality

The codebase for the ERC20MultiDelegate contract is well-structured and adheres to best practices. It leverages OpenZeppelin's libraries for ERC20 and ERC1155 functionalities, which are widely recognized for their reliability and security.

Strengths:
- Well-documented codebase with clear comments and Natspec.
- Consistent use of established libraries for ERC20 and ERC1155 functions.

Weaknesses:
- The code lack proper documentation, this complicated the audit
- The sponsor doesn't answer dms on time, it's crucial for auditors to open private thread instead.

## Security Considerations

The following security considerations were assessed during the audit:

### Proper Permissions and Roles

The contract correctly implements role-based access control, allowing only the owner to change the URI for ERC1155 metadata. This ensures that critical functions are restricted to authorized parties.

### Handling of Input Arrays

The `delegateMulti` function handles input arrays (`sources`, `targets`, and `amounts`) correctly. It verifies that the number of amounts provided is equal to the greater of the number of sources or targets, ensuring that input arrays are consistent.

### Transferring Tokens Between Proxy Delegators

The contract transfers tokens between proxy delegators securely. It checks for available token balances, deploys proxy delegators if necessary, and transfers tokens between delegators with proper accounting. This process is well-structured and does not exhibit vulnerabilities... except for the proxy creation using constant salt with CREATE2 opcod

## Recommendations

Refactor the contract to use a random salt value for deploying new proxies with CREATE2 opcode.This will prevent an attacker from deploying a malicious proxy at the same address and mess up the contract logic 

## Conclusion

The ERC20MultiDelegate contract has been reviewed for code quality, security considerations, and potential vulnerabilities. The contract adheres to best practices, correctly implements permissions and roles, and handles input arrays and token transfers securely. Vulnerabilities or weaknesses were identified during the audit.

This audit report is not a guarantee of absolute security, and ongoing monitoring and testing of the contract are recommended to ensure its continued security.

Please feel free to reach out if you have any questions or require further assistance.

**Note:** This audit report is based on the code provided at the time of the audit and the context provided in the documentation. Any subsequent modifications to the contract may not be covered in this report.

---

### Time spent:
20 hours