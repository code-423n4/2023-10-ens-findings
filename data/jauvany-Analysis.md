# Comments for the judge

**Description**

ENS is a decentralised naming service built on top of Ethereum, and designed to resolve a wide array of resources including blockchain addresses, decentralised content, and user profile information.

**Scope**

> The engagement involved auditing one Solidity Smart Contract: 

- ERC20MultiDelegate.sol: ERC20Votes compatible multi-delegation contract to manage user votings. It uses openzepplin libraries.


# Approach taken in evaluating the codebase

> During this analysis, my main focus was on the multi-delegation mechanism for ERC20 tokens used by ENS.

- Day 1: I spent time understanding the overall working of the ENS system, and getting an overview of the codebase.

- Day 2: I researched possible systemic risks, centralization risks, architecture recommendations, and prepared the final analysis report.


# Architecture recommendations

> ENS has two principal components: the registry, and resolvers.

- Registry: The core contract of ENS, the registry maintains a mapping from domain name to owner, resolver, and time-to-live.

- Resolver: A resolver is a contract that maps from name to the resource. Resolvers are pointed to by the resolver field of the registry.

**Gas Optimizations**

- Struct packing : Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.
 
- Use constant values : If certain values in your contracts are constant and do not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs. 

- Avoid unnecessary storage : Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality. 

- Storage vs. memory usage : When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
 
- Replacing the use of memory with calldata for read-only arguments in external functions .

**Other recommendations**

- Regular code reviews and adherence to best practices. 
- Establish a responsible disclosure policy for vulnerabilities. 
- Implement continuous monitoring for unusual activity. 
- Educate users about risks and best practices.


# Codebase quality analysis

> The overall quality of the codebase for ENS can be classified as ”Good“.

**Strengths**

- The ENS codebase features comprehensive tests, which is a positive indicator of code quality. These tests cover various scenarios and edge cases, helping to ensure that the contracts behave as expected in a wide range of situations. 

- Natspec was really helpful and detailed. 


**Weaknesses**
- Vulnerable versions of packages are being used e.g [CVE-2022-39384](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-39384), [CVE-2022-35961](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-35961), and 
[CVE-2022-35915](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-35915) . Consider switching to more recent versions of these packages that don't have these vulnerabilities.

- Contract uses solidity version ^0.8.2 Adopting a newer version of solidity can be more gas efficient.


# Centralization risks

Centrality risk is present in the contract as the role of onlyOwner detailed below is essential. Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. This may also cause a single point failure.

```solidity
function setUri(string memory uri) external onlyOwner {
```

# Mechanism review

The contract implements a multi-delegation mechanism for ERC20 tokens that support the ERC20Votes extension. The ENS mechanisms include the ERC1155 standard for token metadata. These provide a comprehensive range of features and capabilities.

# Systemic risks

- External Contract Dependencies: The contract relies on  OpenZeppelin’s libraries for standard ERC20 and ERC1155 functionalities. If any of these contracts have vulnerabilities, it would affect this contract.

- Governance Mechanism Security: The contract’s governance mechanism is critical for its operation. A poorly implemented governance mechanism could lead to system-wide issues.

- Like any smart contract-based system, ENS is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.

- Test Coverage: The test coverage provided by ENS is Stmts: 100%, Branch: 91.67%, Funcs: 100%, Lines: 100%, however, I recommend 100% of test coverage for Branch also.

### Time spent:
16 hours