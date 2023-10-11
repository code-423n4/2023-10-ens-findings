# QA Report

## Summary
Total **9 instances** over **7 issues**:

**L**: Low impact issue,
**N**: Non Critical Issues

| ID  | Issue | Instances |
| :---: | :--- | :---: |
| [\[L-01\]](#l-01-delegating-to-address-zero) | Delegating to Address Zero | 1   |
| [\[L-02\]](#l-02-token-transfer-to-erc20proxydelegator) | Token Transfer to `ERC20ProxyDelegator` | 1   |
| [\[N-03\]](#n-03-no-standardized-data-format-for-delegatemulti) | No Standardized Data Format for `delegateMulti` | 1   |
| [\[L-04\]](#l-04-lack-of-a-pausable-mechanism) | Lack of a Pausable Mechanism | 2   |
| [\[L-05\]](#l-05-interaction-with-approved-operators-in-erc1155) | Interaction with Approved Operators in ERC1155 | 1   |
| [\[N-06\]](#n-06-same-source-and-target) | Same Source and Target | 1   |
| [\[N-07\]](#n-07-lack-of-event-emission) | Lack of Event Emission | 2   |
| [\[N-08\]](#n-08-redundant-balance-check-in-_processdelegation) | Redundant Balance Check in `_processDelegation` | 2   |

---

## [L-01] Delegating to Address Zero

The contract allows users to delegate to address zero (0x0), which effectively locks up tokens without a valid delegate. This can be achieved when one of the entities in the `targets` array in the `delegateMulti` params has a value of 0.

**Recommendation**: Add checks to prevent delegating to address zero or implement specific actions.

- [ERC20MultiDelegate.sol#L59](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L59)
  

## [L-02] Token Transfer to `ERC20ProxyDelegator`

Transferring tokens directly to an `ERC20ProxyDelegator` contract before or after the creation is currently counted as a valid vote but is not recoverable. This could lead to unintentional loss of tokens.

**Recommendation**: Implement a mechanism to recover tokens sent directly to `ERC20ProxyDelegator` for improved user experience.

- [ERC20MultiDelegate.sol#L11-#L20](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L11-#L20)
  

## [N-03] No Standardized Data Format for `delegateMulti`

The `delegateMulti` function takes three arrays (sources, targets, and amounts) as arguments, making it less user-friendly and challenging to integrate with other contracts and tools.

**Recommendation**: Consider implementing a structured data format for better usability.

**Code Snippet**:

```solidity
struct Delegation {    
    // maybe type and owner
    address source;
    address target;
    uint256 amount;
}

function delegateMulti(Delegation[] calldata delegations) external {
    // Process delegations using the structured format.
    for (uint256 i = 0; i < delegations.length; i++) {
        _processDelegation(delegations[i]);
    }
    //...
}
```

- [ERC20MultiDelegate.sol#L57-#L61](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-#L61)
  

## [L-04] Lack of a Pausable Mechanism

The contract does not include a pausable mechanism, which can be essential for emergency scenarios, bug fixes, or temporary halting of certain functions.

**Recommendation**: Implement a pausable mechanism for added security and control.

- [ERC20MultiDelegate.sol#L124](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124)
  
- [ERC20MultiDelegate.sol#L155](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155)
  

## [L-05] Interaction with Approved Operators in ERC1155

**Description**: The contract does not allow approved operators in ERC1155 to implement the `delegateMulti` function on behalf of their users. This limitation can impact usability.

**Recommendation**: Explore options to allow approved operators to interact with `delegateMulti`.

- [ERC20MultiDelegate.sol#L57](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57)
  

## [N-06] Same Source and Target

When the source and target addresses are the same, the contract still executes a transfer operation, but this doesn't have any practical impact on the contract's functionality. It results in the emission of redundant events.

**Recommendation**: Optimize the code to avoid redundant event emissions in this scenario.

- [ERC20MultiDelegate.sol#L134](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L134)

## [N-07] Lack of Event Emission

Token transfers from a source delegate (proxy delegator) back to the user in the `_reimburse` function and during the creation of proxy delegators are missing event emissions. This absence of event emission may lead to confusion and misunderstandings about the state of the current situation, as users are not notified when these functionality occur.

**Recommendation**: Implement an event emission mechanism to notify users whenever tokens are transferred during the reimbursement process in the `_reimburse` function and when tokens are sent to proxy delegators. This will improve transparency and user experience.

**Instances**:

- [ERC20MultiDelegate.sol#L144-#L149](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144-#L149) - Lack of event emission during token transfers in the `_reimburse` function.
- [ERC20MultiDelegate.sol#L55-#L161](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-#L161) - Lack of event emission during transfer to proxy delegators.


## [N-08] Redundant Balance Check in `_processDelegation`

In the `_processDelegation` function, there's a redundant balance check on the user's account, ensuring they have a sufficient balance for the token transfer. However, the standard `balanceOf` function check is already performed within the `_burnBatch` function, which is part of the ERC1155 standard behavior.

**Recommendation**: Remove the redundant balance check within `_processDelegation` to optimize gas costs and improve code efficiency.

- [ERC20MultiDelegate.sol#L81-#L83](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L81-#L83)
