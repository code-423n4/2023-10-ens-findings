

Introduction

The Ens Decentralized Naming Service plays a crucial role in the blockchain ecosystem, providing users with a reliable way to manage naming and identity. The objective of this security audit is to identify anomalies and potential security concerns within the service. This report presents two key findings along with recommendations to enhance the security and usability of Ens.

Findings and Recommendations

Issue 1: Conversion of Addresses to uint256 Type

Observation: During the audit, it was observed that the address of delegates is converted to the uint256 type to serve as the ID for a token. While this conversion doesn't affect functionality, it results in most IDs having very large values.

Concerns: The conversion of addresses to uint256 type can lead to issues related to readability and computational overhead. Users may find it challenging to work with such large ID values.

Recommendation: To improve user-friendliness and readability of IDs, we recommend implementing an additional method or mechanism to trim down these IDs.

Issue 2: Deployment of ERC20ProxyDelegator Contract

Observation: The ERC20ProxyDelegator contract is deployed every time a delegation to a new target is involved. This deployment strategy has implications for gas costs and the multi-delegation system's intended purpose.

Concerns: Deploying a new contract for each new delegate increases redundancy on the blockchain and raises the cost of multi-delegating, which contradicts the primary goal of the multi-delegation system—to save gas and combine delegations within a single transaction.

Recommendation: To optimize the multi-delegation system and reduce redundancy on the blockchain, we recommend deploying a single contract to serve as the ERC20ProxyDelegator for both new and existing delegates. This change will significantly reduce gas costs and encourage users to utilize the multi-delegation system more effectively.
In the course of our audit, it's important to note that the Ens Decentralized Naming Service demonstrates strong coding patterns and thoughtful design in various aspects of its implementation. The codebase reflects a commitment to security and functionality, which is commendable. The utilization of decentralized technologies and smart contract principles is evident, and the service's architecture exhibits robustness and reliability. Additionally, the dedication to maintaining the service's decentralized nature is a testament to the development team's commitment to the blockchain ecosystem's core principles. These positive aspects of the coding patterns and design of Ens provide a solid foundation for addressing the identified issues and further enhancing the service's overall performance.


### Time spent:
17 hours