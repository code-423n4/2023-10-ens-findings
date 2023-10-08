# ENS Audit Contest Analysis Report | 5 Oct 2023 - 11 Oct 2023

---

| #   | Topic                                                 |
| --- | ----------------------------------------------------- |
| 1   | ENS Contest Documentation Analysis                    |
| 2   | Codebase Explanation, Examples Executions & Scenarios |
| 3   | Questions & Notes during the Audit                    |
| 4   | My Findings Summary                                   |
| 5   | Potential Attack Vectors Discussed during the Audit   |
| 6   | Time Spent                                            |

---

# 1. ENS Contest Documentation Analysis

**Ethereum Name Service (ENS) Summary**

- **ENS** is a decentralised naming service built on top of Ethereum, and designed to resolve a wide array of resources including blockchain addresses, decentralised content, and user profile information.

- **Purpose**: ENS provides a decentralized naming system on the Ethereum blockchain. It associates human-readable names (e.g., 'alice.eth') with machine identifiers, such as Ethereum addresses, other crypto addresses, content hashes, and more. It can be likened to the Internet's DNS but is uniquely structured to work within Ethereum's capabilities.

- **Functionality**:

  1. Maps names to addresses and other identifiers.
  2. Allows reverse resolution: linking metadata to Ethereum addresses.
  3. Operates with dot-separated hierarchical names or domains. Domain owners have control over their subdomains.
  4. Top-level domains (e.g., '.eth') are governed by smart contracts known as registrars. They set rules for subdomain allocation.
  5. Users can import DNS names they own for use in ENS.
  6. Domain owners can set up and manage subdomains as they see fit.

- **Availability**: Deployed on the Ethereum main network and various test networks. Compatible libraries and apps can recognize and work with the appropriate ENS deployment based on the network they are interacting with.

- **Components**:

  1. **ENS Registry**:

     - A smart contract listing all domains/subdomains.
     - Tracks each domain's owner, resolver, and caching time-to-live (TTL).
     - Domain owners can set resolvers, transfer domain ownership, and manage subdomain ownership.
     - The registry's primary function is to indicate which resolver corresponds to a name.

  2. **Resolvers**:

  - Contracts that translate names into addresses.
  - Resolvers must adhere to standards for the data they manage, e.g., crypto addresses or IPFS content hashes.
  - Resolvers can be general-purpose or specialized based on user needs.
  - Resolving a name involves checking the registry for the appropriate resolver and then querying that resolver.
    ![Alt text](image-2.png)

- **User Interaction**: Users can interact with ENS via the ENS Manager App or through various ENS-enabled applications.

![Alt text](image-1.png)

In the above example, we're trying to find the Ethereum address pointed to by 'foo.eth'. First, we ask the registry which resolver is responsible for 'foo.eth'. Then, we query that resolver for the address of 'foo.eth'.

- **Delegation Mechanics**:

  1. Supports ERC20Votes, allowing for unique delegation capabilities.
  2. Each user-delegate pairing is distinct, ensuring tailored delegation.

- **Underlying Infrastructure**:

  1. **OpenZeppelin Libraries**: The ERC20MultiDelegate.sol contract is built upon OpenZeppelin's foundational libraries which provide functionalities for ERC20 and ERC1155.
  2. **Solidity's Proxy Contracts**: By leveraging Solidity's native proxy contract feature, the system can produce specific delegation capabilities tailored for every user-delegate relationship.

- **Cryptographic and Standard Adherence**:
  1. **No Custom Cryptography**: The contract steers clear from implementing any custom cryptographic algorithms, which is a good security practice.
  2. **ERC Standards**: Utilizes ERC20Votes for managing delegation processes and ERC1155 for token metadata handling.

---

---

# 2. Codebase Explanation, Examples Executions & Scenarios

## 2.1. Purpose and Functionality of `delegateMulti()`:

The `delegateMulti()` function facilitates the process of transferring delegation amounts across multiple source and target delegate pairs. It serves as an interface for the `_delegateMulti()` function, which contains the actual logic.

### Key Steps:

1. **Check Input Validity**:

   - Ensure there's at least one source or target delegate provided.
   - The number of amounts should match the greater number between sources or targets.

2. **Process Delegations**:

   - It iterates over the longer list (sources or targets).
   - For each iteration, it:
     - Sets the source and target addresses.
     - Depending on the current index, transfers delegation from the source to target, or handles leftover amounts from sources or targets.

3. **Token Adjustments**:
   - Burns tokens from the `msg.sender` (the caller) corresponding to sources.
   - Mints new tokens to the `msg.sender` for the targets.

### First Example of Function Execution:

Let's take a hypothetical scenario to better understand its behavior:

Suppose there are 3 source delegates, 2 target delegates, and 3 amounts to transfer:

```solidity
sources = [address1, address2, address3];
targets = [addressA, addressB];
amounts = [100, 50, 75];
```

Execution:

1. It would validate that there's at least one source or target and that the number of amounts (3 in this case) matches the larger list (sources, which has 3 addresses).

2. It will iterate 3 times (the length of the longest list).

   i. For the first iteration:

   - Source: address1
   - Target: addressA
   - Amount: 100
   - Delegation of 100 units will be processed between address1 and addressA.

   ii. For the second iteration:

   - Source: address2
   - Target: addressB
   - Amount: 50
   - Delegation of 50 units will be processed between address2 and addressB.

   iii. For the third iteration:

   - Source: address3
   - No target is provided for this index.
   - Amount: 75
   - Since there's no corresponding target, the amount of 75 units will be reimbursed back to the caller from address3.

3. The `msg.sender` will have 225 (100 + 50 + 75) units burned because of the source addresses and 150 (100 + 50) units minted because of the target addresses.

### Second Example of Function Execution:

```
sources = [address1, address1, address3];
targets = [addressA, addressA];
amounts = [100, 50, 75];
```

Here's a step-by-step breakdown:

1. `sourcesLength = 3`, `targetsLength = 2`, `amountsLength = 3`.

2. The function checks that the `amounts` array has the same length as the greater of `sources` and `targets`, which it does in this case. So, no `require` statement is triggered.

3. The function will now iterate 3 times since `Math.max(sourcesLength, targetsLength) = 3`.

4. In the first iteration:

   - `source` is `address1`
   - `target` is `addressA`
   - `amount` is `100`
   - The `_processDelegation()` function is called, and 100 tokens are transferred from `address1` to `addressA`.

5. In the second iteration:

   - `source` is `address1`
   - `target` is `addressA`
   - `amount` is `50`
   - The `_processDelegation()` function is called again, and 50 more tokens are transferred from `address1` to `addressA`.

6. In the third iteration:
   - `source` is `address3`
   - `target` is `address(0)` (i.e., the zero address) because the `targets` array has only 2 elements.
   - `amount` is `75`
   - Since the `targets` array is exhausted but there are still sources left, the `_reimburse()` function is called. This function essentially reimburses `address3` with 75 tokens.

### Third Example of Function Execution:

```
sources = [address1, address1, address3];
targets = [addressA, addressB];
amounts = [100, 50, 75];
```

This case will proceed similarly to _Second Example_ with one key difference:

1. In the second iteration:
   - `source` is `address1`
   - `target` is `addressB`
   - `amount` is `50`
   - The `_processDelegation()` function is called, transferring 50 tokens from `address1` to `addressB`.

The third iteration remains the same as in _Second Example_, where `address3` is reimbursed with 75 tokens.

## 2.2. Explanation of `retrieveProxyContractAddress()` function:

- The `retrieveProxyContractAddress()` function appears to compute the address of a contract before it is actually deployed on the Ethereum blockchain. This is commonly known as the "create2 address".

1. **Inputs**:
   - `_token`: An instance or the address of an `ERC20Votes` token.
   - `_delegate`: The address of a delegate.
2. **Process**:
   - It starts by creating the `bytecode` for a new contract (`ERC20ProxyDelegator`). The bytecode consists of the contract's `creationCode` concatenated with the ABI-encoded parameters `_token` and `_delegate`.
   - It then computes a `hash` using the keccak256 hashing function. The input to this hash function includes:
     - `bytes1(0xff)`: A constant prefix.
     - `address(this)`: The address of the current contract (the contract from which `retrieveProxyContractAddress` is called).
     - `uint256(0)`: A salt value, which is set to 0 here. This salt allows the generation of different addresses based on different inputs even when using the same bytecode. However, in this case, the salt is static and always 0.
     - `keccak256(bytecode)`: A hash of the previously mentioned bytecode.
3. **Output**:
   - Finally, the function returns the address derived from the hash. This address will be where the `ERC20ProxyDelegator` contract will reside if deployed using the provided `_token` and `_delegate` parameters with the specified creation code and the given salt.

**Purpose**:
This function is used to determine the address of a proxy contract that represents a delegate's voting power in an ERC20Votes system before it's actually deployed.

## 2.3. Explanation of `deployProxyDelegatorIfNeeded()` function **execution**

1. **Get Predicted Proxy Address**:

   ```solidity
   address proxyAddress = retrieveProxyContractAddress(token, delegate);
   ```

   This line calls the `retrieveProxyContractAddress()` function to get the expected address of the `ERC20ProxyDelegator` contract for the given `token` and `delegate` combination. At this point, the contract might not have been deployed yet, but we can predict its address because of how Ethereum's `CREATE2` opcode works.

2. **Check If The Proxy Contract Is Already Deployed**:

   ```solidity
   uint bytecodeSize;
   assembly {
       bytecodeSize := extcodesize(proxyAddress)
   }
   ```

   Here, the code uses inline assembly to check the size of the bytecode at the predicted address. The `extcodesize` opcode in the Ethereum Virtual Machine (EVM) returns the size of the bytecode of the contract deployed at the given address. If there's no contract at that address, it returns `0`.

3. **Deploy the Proxy Contract If Not Already Deployed**:

   ```solidity
   if (bytecodeSize == 0) {
       new ERC20ProxyDelegator{salt: 0}(token, delegate);
       emit ProxyDeployed(delegate, proxyAddress);
   }
   ```

   If `bytecodeSize` is `0`, it means the `ERC20ProxyDelegator` contract hasn't been deployed at the predicted address. In that case, the code deploys the contract using the `new` keyword with the given `token` and `delegate` as arguments. The `{salt: 0}` syntax specifies a salt of `0` for the deployment, which matches what was used in the `retrieveProxyContractAddress` function. Once the contract is deployed, an event `ProxyDeployed` is emitted, notifying listeners of the new contract's address.

4. **Return the Proxy Address**:
   ```solidity
   return proxyAddress;
   ```

**In summary**, this function ensures that a unique `ERC20ProxyDelegator` contract exists for each `token` and `delegate` combination. If it doesn't exist, it deploys one, and then returns the address of the proxy contract.

---

---

# 3. Questions & Notes during the Audit

### 3.1. What the "delegate pairs" and "delegation amounts" means?

- The terms "delegate pairs" and "delegation amounts" refer to the mechanism of assigning certain amounts of tokens (or voting power) from one address to another. Let's break down these terms:

1.  **Delegate Pairs**:

- This typically refers to a pair of addresses consisting of a "source" and a "target".
- The "source" is the address from which tokens or voting power will be taken or withdrawn.
- The "target" is the address to which these tokens or voting power will be assigned or transferred.
- In the case of the `delegateMulti()` function, you can provide multiple source and target addresses, meaning you can perform multiple delegation transfers in a single transaction.

2.  **Delegation Amounts**:

- This refers to the specific amount of tokens or voting power being transferred from a source delegate to a target delegate.
- The `delegateMulti()` function allows for multiple amounts to be specified, one for each delegate pair.
- The amounts denote how much is being delegated from the respective source to its corresponding target.

### 3.2. What actually this sentence means: "@dev: A utility contract to let delegators pick multiple delegates"?

1. **Utility Contract**: This term typically refers to a smart contract that provides a specific function or set of functions. Instead of being a full-fledged application or protocol, utility contracts usually offer auxiliary services or functions that other contracts or users can utilize. In essence, it's a tool or helper contract.
2. **Delegators**: Delegators are entities (typically, but not always, addresses on a blockchain) that assign or "delegate" some authority, responsibility, or asset to another entity. In many blockchain contexts, delegation allows one address to give another address the ability to represent it in some manner, such as voting.
3. **Pick Multiple Delegates**: This suggests that the utility contract allows a delegator to select more than one delegate. In other words, instead of a one-to-one delegation (one delegator assigning their authority to one delegate), the utility contract enables a one-to-many delegation (one delegator assigning their authority to multiple delegates).

- In summary, the sentence describes a smart contract that provides a function for users to delegate authority or responsibility to multiple other users at once.

### 3.3 Why EIP1155 standard is used?

- EIP1155 is used to keep track of all delegates for a token holder. A token holder will call `delegateMultifunction` to delegate his token, this will trigger a proxy deployment (if needed) for each delegate the token holder has decided to delegate to. The token holder will send his token to the proxy (or from a proxy to another, but this is still tokens actually owned by the token holder), and ERC20MultiDelegate inherits from ERC1155 to keep track of delegated tokens actually owned by the token holder, in all different proxy delegate contracts. Here, ID of erc1155 `_balances` mapping is the delegate address (and not a token). `_burnBatch` and `_mintBatch` are here used to update ERC1155 mappings and check if the caller of (delegateMulti) is legitimate.

---

---

# 4. My Findings Summary

### 4.1 QA Report:

| ID              | Title                                                                                                                                      | Instances | Severity                  |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------- | ------------------------- |
| [L-01](#L-01)   | Unbounded `for` loop in `_delegateMulti()`. Set max value for the loop iterations (max values for `sourcesLength[]` and `targetsLength[]`) | 1         | _Low_                     |
| [L-02](#L-02)   | Static Salt in `deployProxyDelegatorIfNeeded()`: Possibility of DoS due to Predictable Contract Address                                    | 1         | _Low_                     |
| [L-03](#L-03)   | Unchecked Initialization in `deployProxyDelegatorIfNeeded()`: Risk of DoS due to Potential Malfunction in `ERC20ProxyDelegator`            | 1         | _Low_                     |
| [L-04](#L-04)   | Front Running of `ERC20ProxyDelegator` Deployment                                                                                          | 1         | _Low_                     |
| [L-05](#L-05)   | Gas Concerns - DoS in `deployProxyDelegatorIfNeeded()`                                                                                     | 1         | _Low_                     |
| [NC-01](#NC-01) | Potential Reversion Issue with Certain ERC20 Token Approvals                                                                               | 1         | _Non Critical_            |
| [NC-02](#NC-02) | Unchecked Return Values for `approve()`                                                                                                    | 1         | _Non Critical_            |
| [NC-03](#NC-03) | Need for Comments on State Variables                                                                                                       | 1         | _Non Critical_            |
| [NC-04](#NC-04) | Absence of Event Emissions                                                                                                                 | 3         | _Non Critical_            |
| [NC-05](#NC-05) | Missing address zero checks for `sources[]` and `targets[]` arrays in `delegateMulti()` function                                           | 1         | _Non Critical_            |
| [S-01](#S-01)   | Optimization for `deployProxyDelegatorIfNeeded()` function logic                                                                           | -         | _Suggestion/Optimization_ |
| [S-02](#S-02)   | Optimization for transferring flow                                                                                                         | -         | _Suggestion/Optimization_ |

---

---

# 5 Potential Attack Vectors Discussed during the Audit

1. Check for proper permissions and roles.
2. Ensure that the delegateMulti function handles array inputs correctly.
3. Validate the logic for transferring between proxy delegators.
4. Tokens should only be transferred between approved delegators.

---

---

# 6. Time Spent

Day 1:

- Investigated the ENS Protocol.
- Reviewed the ENS Audit Contest Documentation and the ERC20MultiDelegate.sol contract codebase.
- Wrote example Executions and Scenarios for the ERC20MultiDelegate.sol contract codebase logic and behavior.
- Found several potential Attack Vectors.
- Began writing the Analysis report.
- Began writing the QA report.

Day2:

- Took a deep dive into the ERC1155 and ERC20Votes standards.

### Time spent:
30 hours