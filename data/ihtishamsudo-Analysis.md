## Analysis of Codebase 
I'll try to give overall analysis about the project and codebase in scope.
### [1.1] General Intrduction About ENS 
To understand contract or codebase in scope we need to know about the general view of the protocol
- Brief intro about ENS 
The Ethereum Name Service (ENS) stands as a decentralized and adaptable naming platform, operating on the Ethereum blockchain. Its primary function involves translating user-friendly names like 'alice.eth' into technical identifiers like Ethereum addresses, various cryptocurrency addresses, content hashes, and metadata. Additionally, ENS facilitates 'reverse resolution,' enabling the linkage of metadata such as official names or interface details with Ethereum addresses.

Here it was the general intro about what ENS is. 

I'll try to give a short introduction about ```ERC20MultiDelegate``` (Current codebase in scope). 

- Brief intro about ```ERC20MultiDelegate``` (Codebase in scope) 

The current Codebase in scope contains ```ERC20MultiDelegate``` contract:

ERC20MultiDelegate contract manages the delegation process efficiently and securely. Together, they enable token holders to delegate tokens to multiple delegates, handle transfers, create proxy contracts, and ensure that unused tokens are returned to the delegators.

### [1.2] Analysis Of Contract(s) in scope 

The current codebase contains four contracts but there is only one contract in scope for this audit so I'll try to explain its functionality and process flow of this contract.

Current Codebase in scope contains contract that introduces two main components:

1. **ERC20MultiDelegate**: This contract extends ERC1155 and Ownable contracts. It allows delegators to split their tokens among multiple delegates. Delegators can specify various source delegates, target delegates, and corresponding amounts to transfer. The contract ensures secure token transfers and emits events (`ProxyDeployed` and `DelegationProcessed`) for tracking these transactions. Delegation transfers can be initiated by calling the `delegateMulti` function.

2. **ERC20ProxyDelegator**: This contract is a child contract deployed by ERC20MultiDelegate. It acts as a proxy delegator to vote on behalf of the original delegator. It is constructed with an ERC20Votes token and the target delegate address, allowing efficient delegation of voting power.

I find contract structure very interesting and will try to break it down in minimalist words to make this codebase understandable. 

Certainly! Let's break down the workflow of the `delegateMulti` and `_delegateMulti` functions and explain their purpose step by step:

### `delegateMulti` Function:

1. **Function Signature and Parameters:**
   ```solidity
   function delegateMulti(
       uint256[] calldata sources,
       uint256[] calldata targets,
       uint256[] calldata amounts
   ) external {
   ```
   The `delegateMulti` function takes three arrays as input parameters: `sources`, `targets`, and `amounts`. These arrays represent the source delegates, target delegates, and corresponding amounts of tokens to be transferred between them, respectively. The function is declared as `external`, allowing it to be called from outside the contract.

2. **Delegation Validation:**
   ```solidity
   require(
       sources.length > 0 || targets.length > 0,
       "Delegate: You should provide at least one source or one target delegate"
   );
   require(
       Math.max(sources.length, targets.length) == amounts.length,
       "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
   );
   ```
   The function performs validation checks to ensure that valid delegation data is provided. It requires at least one source or one target delegate to be provided, and it checks that the number of amounts corresponds to the maximum of the number of sources or targets.

3. **Delegation Process Loop:**
   ```solidity
   for (uint transferIndex = 0; transferIndex < Math.max(sources.length, targets.length); transferIndex++) {
   ```
   The function enters a loop that iterates through the provided source and target delegates.

4. **Processing Delegation Transfers:**
   ```solidity
   if (transferIndex < Math.min(sources.length, targets.length)) {
       _processDelegation(source, target, amount);
   } else if (transferIndex < sources.length) {
       _reimburse(source, amount);
   } else if (transferIndex < targets.length) {
       createProxyDelegatorAndTransfer(target, amount);
   }
   ```
   - If `transferIndex` is within the range of valid source and target delegates, `_processDelegation` function is called to transfer tokens between the current source and target delegate pair.
   - If `transferIndex` is within the range of only sources, any remaining source amounts after the transfer process are handled by calling the `_reimburse` function.
   - If `transferIndex` is within the range of only targets, any remaining target amounts after the transfer process are handled by creating a proxy delegator contract and transferring tokens using `createProxyDelegatorAndTransfer` function.

5. **Finalizing the Delegation Process:**
   ```solidity
   if (sources.length > 0) {
       _burnBatch(msg.sender, sources, amounts[:sources.length]);
   }
   if (targets.length > 0) {
       _mintBatch(msg.sender, targets, amounts[:targets.length], "");
   }
   ```
   After processing all delegation transfers, the function finalizes the process by burning the token amounts from the sender for the source delegates and minting the token amounts to the sender for the target delegates. This ensures that the correct token balances are adjusted based on the delegation transfers.

### `_delegateMulti` Function (internal):

The `_delegateMulti` function is an internal helper function that performs similar actions to the `delegateMulti` function. It is used internally within the contract to handle the logic of delegation transfers. The primary purpose of splitting the logic into an external and an internal function is to maintain a clean and modular code structure, allowing for easier testing and readability.

```delegatemulti``` functions enable the contract to efficiently handle multiple delegation transfers, ensuring that tokens are transferred correctly between source and target delegates while maintaining proper accounting of token balances.

### `_processDelegation` Function:

This internal function is responsible for processing the delegation transfer between a source delegate and a target delegate. Here's a step-by-step explanation of its workflow:

1. **Input Parameters:**
   ```solidity
   function _processDelegation(
       address source,
       address target,
       uint256 amount
   ) internal {
   ```
   The function takes three parameters: `source` (the source delegate from which tokens are being withdrawn), `target` (the target delegate to which tokens are being transferred), and `amount` (the amount of tokens to be transferred between the source and target delegates).

2. **Check Delegate Balance:**
   ```solidity
   uint256 balance = getBalanceForDelegate(source);
   assert(amount <= balance);
   ```
   The function first checks the balance of the source delegate to ensure it has sufficient tokens (`balance >= amount`) for the delegation transfer. The `getBalanceForDelegate` function is called to retrieve the balance of the source delegate. If the source delegate does not have enough balance, the `assert` statement will revert the transaction, ensuring the function fails gracefully if the condition is not met.

3. **Deploy Proxy Delegator (if needed):**
   ```solidity
   deployProxyDelegatorIfNeeded(target);
   ```
   The function calls the `deployProxyDelegatorIfNeeded` internal function, which ensures that a proxy delegator contract is deployed for the target delegate if it hasn't been deployed already. Proxy delegators are used to facilitate voting or other actions on behalf of the delegators in a more gas-efficient manner.

4. **Transfer Tokens Between Delegators:**
   ```solidity
   transferBetweenDelegators(source, target, amount);
   ```
   The function then calls the `transferBetweenDelegators` internal function, which transfers the specified `amount` of tokens from the source delegate to the target delegate. This function handles the actual token transfer between the delegators.

5. **Emit Event:**
   ```solidity
   emit DelegationProcessed(source, target, amount);
   ```
Function emits a `DelegationProcessed` event. Events are useful for tracking and logging significant state changes on the blockchain. In this case, the event indicates that a delegation transfer has been successfully processed between the specified source and target delegates with the specified amount of tokens.

The `_processDelegation` function ensures the validity of the delegation transfer by checking the source delegate's balance, deploys a proxy delegator for the target delegate if necessary, transfers tokens between the source and target delegates, and emits an event to log the processed delegation transfer.

### `_reimburse` Function:

This internal function is responsible for reimbursing any remaining token amounts back to the delegator after the delegation transfer process. Here's a step-by-step explanation of its workflow:

1. **Input Parameters:**
   ```solidity
   function _reimburse(address source, uint256 amount) internal {
   ```
   The function takes two parameters: `source` (the source delegate from which tokens are being withdrawn) and `amount` (the amount of tokens to be withdrawn from the source delegate).

2. **Retrieve Proxy Contract Address:**
   ```solidity
   address proxyAddressFrom = retrieveProxyContractAddress(token, source);
   ```
   The function calls the `retrieveProxyContractAddress` internal function to obtain the proxy contract address associated with the specified `source` delegate. Proxy contracts are used to efficiently manage token transfers and voting processes. The retrieved `proxyAddressFrom` represents the source delegate's proxy contract address.

3. **Transfer Tokens to Delegator:**
   ```solidity
   token.transferFrom(proxyAddressFrom, msg.sender, amount);
   ```
   Using the `transferFrom` function of the `token` (an ERC20Votes contract), the function transfers the specified `amount` of tokens from the `proxyAddressFrom` (source delegate's proxy contract) to the `msg.sender`. Here, `msg.sender` represents the delegator who initiated the reimbursement.

   This step ensures that any remaining tokens from the source delegate are returned to the delegator after the delegation transfer process. If there are no remaining tokens (`amount` equals the remaining balance), the full source amount is transferred back to the delegator.

The `_reimburse` function allows the contract to handle the return of remaining tokens from a source delegate back to the delegator. It ensures that the delegator receives any remaining tokens that were not transferred during the delegation process, maintaining accurate token balances for both the delegator and the source delegate.

### `setUri` Function:

```solidity
function setUri(string memory uri) external onlyOwner {
    _setURI(uri);
}
```

**Purpose:**
The `setUri` function allows the owner of the contract to set the metadata URI associated with the ERC1155 tokens. Metadata URI provides a way to fetch additional off-chain information about the tokens, such as their images, descriptions, or other attributes.

**Workflow:**
1. **Access Control:**
   - The function is marked as `external`, meaning it can be called from outside the contract.
   - The `onlyOwner` modifier ensures that only the owner of the contract (the one who deployed it) can invoke this function.

2. **Setting Metadata URI:**
   - The function calls the `_setURI` internal function, passing the provided `uri` parameter. `_setURI` is likely a function defined elsewhere in the contract, responsible for setting the metadata URI for the ERC1155 tokens.

### `createProxyDelegatorAndTransfer` Function:

```solidity
function createProxyDelegatorAndTransfer(
    address target,
    uint256 amount
) internal {
    address proxyAddress = deployProxyDelegatorIfNeeded(target);
    token.transferFrom(msg.sender, proxyAddress, amount);
}
```

**Purpose:**
The `createProxyDelegatorAndTransfer` function is responsible for creating a proxy delegator contract for a target delegate (if necessary) and transferring a specified amount of tokens to that proxy delegator.

**Workflow:**
1. **Deploy Proxy Delegator:**
   - The function calls `deployProxyDelegatorIfNeeded(target)` to check if a proxy delegator contract already exists for the specified `target` delegate. If not, it deploys a new proxy delegator contract for the `target` delegate.

2. **Token Transfer:**
   - The function transfers `amount` tokens from the sender (`msg.sender`) to the `proxyAddress`, which represents the proxy delegator contract created for the `target` delegate.
   - This step effectively transfers tokens from the delegator to the proxy delegator contract, allowing the proxy to perform actions on behalf of the delegator.

### `transferBetweenDelegators` Function:

```solidity
function transferBetweenDelegators(
    address from,
    address to,
    uint256 amount
) internal {
    address proxyAddressFrom = retrieveProxyContractAddress(token, from);
    address proxyAddressTo = retrieveProxyContractAddress(token, to);
    token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
}
```

**Purpose:**
The `transferBetweenDelegators` function facilitates the transfer of tokens between two proxy delegator contracts. It ensures that tokens are moved from the `from` proxy to the `to` proxy.

**Workflow:**
1. **Proxy Address Retrieval:**
   - The function retrieves the proxy contract addresses for both the `from` and `to` delegates using the `retrieveProxyContractAddress` internal function.

2. **Token Transfer:**
   - It then transfers `amount` tokens from the `proxyAddressFrom` (proxy of the `from` delegate) to the `proxyAddressTo` (proxy of the `to` delegate).
   - This transfer ensures that tokens are moved between the two proxy delegator contracts, allowing for seamless delegation of tokens from one delegate to another.

These functions work together to manage the delegation process efficiently. `setUri` allows the owner to set metadata URI, `createProxyDelegatorAndTransfer` creates proxy delegators and transfers tokens to them, and `transferBetweenDelegators` moves tokens between proxy delegator contracts, ensuring smooth token delegation and management.

### `deployProxyDelegatorIfNeeded` Function:

```solidity
function deployProxyDelegatorIfNeeded(
    address delegate
) internal returns (address) {
    address proxyAddress = retrieveProxyContractAddress(token, delegate);

    // check if the proxy contract has already been deployed
    uint bytecodeSize;
    assembly {
        bytecodeSize := extcodesize(proxyAddress)
    }

    // if the proxy contract has not been deployed, deploy it
    if (bytecodeSize == 0) {
        new ERC20ProxyDelegator{salt: 0}(token, delegate);
        emit ProxyDeployed(delegate, proxyAddress);
    }
    return proxyAddress;
}
```

**Purpose:**
The `deployProxyDelegatorIfNeeded` function is responsible for deploying a proxy delegator contract for a specified delegate if it hasn't been deployed already. Proxy delegators are used to handle token delegation efficiently.

**Workflow:**
1. **Proxy Contract Retrieval:**
   - The function first retrieves the proxy contract address associated with the specified `delegate` by calling `retrieveProxyContractAddress(token, delegate)`.

2. **Check Proxy Contract Existence:**
   - Using inline assembly, the function checks if the `proxyAddress` contract already has code deployed (`extcodesize(proxyAddress)`). If the `bytecodeSize` is 0, it means the contract does not exist.

3. **Deploy Proxy Contract:**
   - If the proxy contract doesn't exist, a new `ERC20ProxyDelegator` contract is deployed with the specified `token` and `delegate` parameters.
   - The `salt: 0` parameter ensures that each deployment creates a unique contract instance.
   - An `emit` statement logs the deployment event with the `delegate` and `proxyAddress` details.

4. **Return Proxy Address:**
   - The function returns the `proxyAddress`, whether it was newly deployed or already existed. This address can be used for subsequent token transfers and delegations.

### `getBalanceForDelegate` Function:

```solidity
function getBalanceForDelegate(
    address delegate
) internal view returns (uint256) {
    return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
}
```

**Purpose:**
The `getBalanceForDelegate` function is used to retrieve the balance of a specific delegate for the ERC1155 tokens.

**Workflow:**
1. **Balance Retrieval:**
   - The function calls the `balanceOf` function of the ERC1155 contract (this contract) to retrieve the balance of the `msg.sender` (caller) for the specified `delegate`.
   - The `uint256(uint160(delegate))` conversion is likely a way to convert the `delegate` address into a format suitable for ERC1155 token IDs.

2. **Return Balance:**
   - The function returns the balance of the `delegate` for the caller. This balance represents the number of tokens the caller has for the specified delegate.

### `retrieveProxyContractAddress` Function:

```solidity
function retrieveProxyContractAddress(
    ERC20Votes _token,
    address _delegate
) private view returns (address) {
    bytes memory bytecode = abi.encodePacked(
        type(ERC20ProxyDelegator).creationCode, 
        abi.encode(_token, _delegate)
    );
    bytes32 hash = keccak256(
        abi.encodePacked(
            bytes1(0xff),
            address(this),
            uint256(0), // salt
            keccak256(bytecode)
        )
    );
    return address(uint160(uint256(hash)));
}
```

**Purpose:**
The `retrieveProxyContractAddress` function generates the address of a proxy contract for a specified `_delegate` and `_token`. It follows a specific calculation pattern to ensure each proxy contract has a unique address.

**Workflow:**
1. **Creation Code Encoding:**
   - The function encodes the creation code of the `ERC20ProxyDelegator` contract along with the provided `_token` and `_delegate` parameters.
   - This encoded data includes the contract type, `_token`, and `_delegate` information.

2. **Address Calculation:**
   - The function calculates the contract address by hashing the encoded creation code, the contract creator's address (`address(this)`), and a salt value of `0`. The salt value is added to ensure unique contract addresses for each deployment.
   - The result is a `bytes32` hash.

3. **Conversion and Return:**
   - The function converts the `bytes32` hash to `uint256` and then to `address` using the `uint160(uint256(hash))` method.
   - The resulting address represents the unique proxy contract address for the specified `_token` and `_delegate`.

These functions work together to manage the deployment and retrieval of proxy delegator contracts, ensuring each delegate has a distinct proxy for efficient token delegation and management.

### [1.3] Architectural Improvement 

#### [1.3.1] Switch to Foundry For Better Fuzzing and Testing Coverage 
- Current codebase contains ```Hardhat Testing```
- Hardhat testing is good for unit testing but it does not provide fuzzing and invariant testing which uncover the edge cases and critical state changes that a function can go thru.
- Switiching to foundry will give advantage of fuzzing and invariant testing which make the code more secure by exposing all the uncover or hidden functions states that we can't test with hardhat.

#### [1.3.2] Improved Documentation 
- Code comments were the only documentation for the current codebase in scope. IMO, documentation should be provided by enough reputable Protocol for every codebase or contract(s) so auditoor can understand the codebase properly and can find ways to break things properly.
- unclear or equal to none documentation will lead auditoor to assume false assumptions because of lack of documentation.

### [1.4] Codebase Quality 

**Codebase Quality Analysis Report**

**Overall Rating:** 50%

---

**1. **Documentation Analysis:**
   - **Issue:** The codebase lacks comprehensive documentation, making it difficult for developers and auditors to understand the contracts' functionality, usage, and intended behavior.
   - **Recommendation:** Improve documentation by adding clear and extensive inline comments throughout the code. Develop high-level documentation explaining the contract architecture, data flow, and external interactions. Document function parameters, return values, and potential exceptions. Provide usage examples for essential functions.

---

**2. **Fuzzing Analysis:**
   - **Issue:** Fuzz testing, a critical technique for identifying vulnerabilities, has not been implemented. This leaves the system susceptible to unforeseen inputs and potential exploits.
   - **Recommendation:** Integrate fuzz testing into the testing strategy. Utilize tools like Echidna, foundry or other fuzzing frameworks to generate random, invalid, and unexpected inputs. Focus on critical functions, including token transfers and delegation processes. Regularly run fuzz tests to discover vulnerabilities related to unexpected inputs.

---

**3. **Invariant Testing Analysis:**
   - **Issue:** Invariant testing, which ensures critical properties remain true under all conditions, is missing in the codebase. This oversight can lead to unexpected behaviors and vulnerabilities.
   - **Recommendation:** Identify essential invariants within the contracts, such as token balances never turning negative and consistent state transitions. Develop specific test cases to validate these invariants. Implement both positive and negative scenarios to thoroughly assess the contracts' behavior under various conditions. Document these invariants and associated tests for future reference and auditing purposes.

## [1.4] Codebase Quality Analysis 

The `setUri` function in the provided code is exposed to centralization risk due to its reliance on the `onlyOwner` modifier. Let's break down why this function poses a centralization risk:

```solidity
function setUri(string memory uri) external onlyOwner {
    _setURI(uri);
}
```

**Explanation:**

1. **Access Restriction:** The `setUri` function is marked as `external`, meaning it can be called from outside the contract by anyone. However, the function is protected by the `onlyOwner` modifier, restricting its use to the owner of the contract.

2. **Ownership Control:** The `onlyOwner` modifier restricts the function to be executed only by the owner of the contract, as defined in the `Ownable` contract (presumably used in the contract's code, although it's not explicitly shown in the provided snippet).

**Centralization Risk:**

The centralization risk arises from the fact that the ability to change the URI (Uniform Resource Identifier) of the contract's metadata is solely controlled by the owner.

### [1.5] Time Spent on this codebase 
1-3 Hours : Overview of Codebase
3-5 Hours : Understanding concepts of Core codebase
5-7 Hours : Reading Test and Finding Weak Spots 
7-10 Hours : Writing Analysis 

Total Time Spent : 10 Hours 


### Time spent:
10 hours