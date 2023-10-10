# QA Report for ENS

## Overview

During the audit, 2 Non-Critical and 3 Low issues were found.

### Low-Risk Issues

| # | Issue |
| --- | --- |
| [L-01](#l-01-safetransferfrom-and-safebatchtransferfromshould-be-disabled) | safeTransferFrom and safeBatchTransferFromshould be disabled |
| [L-02](#l-02-implementing-a-maxlimit-for-delegatemulti) | Implementing a maxLimit for delegateMulti |
| [L-03](#l-03-using-a-dynamic-salt-value) | Using a Dynamic salt Value |

### Non-Critical Issues

| # | Issue |
| --- | --- |
| [NC-01](#nc-01-switching-from-assert-to-require) | Switching from assert() to require() |
|[NC-02](#nc-02-lack-of-events-for-critical-operations)|Lack of Events for critical operations|

## Low-Risk Issues

### [L-01] `safeTransferFrom` and `safeBatchTransferFrom`should be disabled

**Description**

- The current implementation of the protocol inherits ERC1155 to manage delegation balances by minting and burning tokens.
- However, this approach poses potential risks, including the loss of ERC20Votes tokens if ERC1155 tokens are inadvertently sent to an unknown address through the **`safeTransferFrom`** function.

**Recommendation**

- To mitigate these risks, we recommend overriding the **`safeTransferFrom`** and **`safeBatchTransferFrom`** functions to ensure that they revert when called.
- This proactive measure will protect users from unintended consequences and enhance the security of the protocol.
    
```solidity
function safeTransferFrom(
    address from,
    address to,
    uint256 id,
    uint256 amount,
    bytes memory data
    ) public virtual override{
    
    	revert("Operation Not supported!");
}
    
function safeBatchTransferFrom(
    address from,
    address to,
    uint256[] memory ids,
    uint256[] memory amounts,
    bytes memory data
    ) public virtual override {
    
    	revert("Operation Not supported!");
}
```
 

### [L-02] **Implementing a `maxLimit` for `delegateMulti`**

**Description**

- In the current system, users can perform multiple actions within a single transaction using the **`delegateMulti`** function found in **`ERC20MultiDelegate.sol`**.
- While this functionality offers convenience, it can lead to transaction failures due to Out-of-Gas errors when large arrays are supplied as inputs.

**Recommendation**

- To optimize the operation and prevent potential Out-of-Gas errors, it is advisable to introduce a **`maxLimit`** variable.
- This variable will limit the size of inputs that can be processed within a single transaction. Here is a modified code snippet illustrating the implementation:
    
    ```solidity
    uint16 public maxLimit; 
    
    function delegateMulti(
            uint256[] calldata sources,
            uint256[] calldata targets,
            uint256[] calldata amounts
        ) external {
            require(amounts.length <= maxLimit, "Input size exceeds maxLimit");
            _delegateMulti(sources, targets, amounts);
     }
    ```
    

### [L-03] **Using a Dynamic `salt` Value**

**Description**

- In the current system, the salt value used while deploying and calculating the address for the **`ERC20ProxyDelegator`** for a user is hardcoded as **`0`**. This practice can lead to potential issues and reduce the system's robustness.

**Recommendation**

- To enhance the robustness of the system, it is advisable to calculate the salt value dynamically as follows:

```solidity
uint256 salt = uint256(keccak256(abi.encodePacked(delegate)));
```

- This change ensures that each deployment of **`ERC20ProxyDelegator`** has a unique salt value based on the **`delegate`** parameter. This approach helps prevent potential collisions or issues that may arise from using a hardcoded salt value.

## Non-Critical Issues

### [NC-01] Switching from `assert()` to `require()`

**Description**

- In the contract at [Line 131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131), the **`assert()`** function is used to check a condition.
- However, using **`assert()`** consumes all remaining gas and reverts all changes made to the contract when the condition evaluates to false.
- It's recommended to use **`require()`** instead, as it also reverts changes but refunds unused gas fees, which is a safer and more gas-efficient approach.

**Recommendation**

- Replace the **`assert()`** statement with a **`require()`** statement at [Line 131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131) as follows:
    
```diff
- assert(amount <= balance);
+ require(amount <= balance, "Amount exceeds balance");
```
    
- By making this change, you ensure that the contract behaves more predictably, consumes gas efficiently, and provides informative error messages in case of failed conditions.

### [NC-02] Lack of Events for critical operations

**Description**

- The `ERC20MultiDelegate` smart contract performs critical operations such as `reimburse`, `transferBetweenDelegators`, `createProxyDelegatorAndTransfer`. But these operations lack `events`.

**Recommendation**
- Hence it is recommended to emit events when above operations are successfully executed for off-chain analysis.