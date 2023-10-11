# QA Report

## **[L-01]** - the token state variable can be declared as `immutable`

## Summary
The token state variable can be declared as `immutable` since there is no possibility of the ENS contract address being updated in the future. This declaration can lead to gas savings.

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L28

## Tools Used
Manual Review

## Remediation Steps

Declare the ENS token as `immutable`:

```
ERC20Votes public immutable token;
```
---

## **[L-02]** - Limit the Size of Input Array in `delegateMulti` to Prevent Out-of-Gas Issues

## Summary
The current implementation of the `delegateMulti` function does not have a check to limit the size of the input array, which can result in potential out-of-gas transactions when users stake a large number of ENS tokens to numerous delegates.

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L62

## Tools Used
Manual Review

## Remediation Steps
To address this issue, it's recommended to set an upper limit on the size of the input array based on thorough testing and revert the transaction if it exceeds the defined limit. This will help prevent out-of-gas issues.

---

## **[L-03]** - Implement Sanity Checks for Zero-Value Inputs in `delegateMulti` Array

## Summary
The current implementation of the `delegateMulti` function lacks validation for zero-value inputs in the array. Two examples that illustrate this are users delegating tokens to a zero address and users passing a zero value as an amount. While there may not be a direct security risk, it is advisable to implement these sanity checks to ensure robustness and user-friendliness.

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98-L107

## Tools Used
Manual Review

## Remediation Steps
Implement basic sanity checks in the `_processDelegation`, `_reimburse`, and `createProxyDelegatorAndTransfer` functions to verify zero-value inputs.

---

## **[L-04]** - Add Balance Check in `_reimburse` Similar to `_processDelegation`

## Summary
The `_processDelegation` function includes a check on the source's ERC1155 token balance before transferring tokens to the target. However, this same check is missing in the `_reimburse` function.

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144

## Tools Used
Manual Review

## Remediation Steps

Add a balance check in the `_reimburse` function, similar to the one in `_processDelegation`.

```
function _reimburse(address source, uint256 amount) internal {
  uint256 balance = getBalanceForDelegate(source);
  require(amount <= balance, "Insufficient balance for reimbursement");

  // Rest of the code
}
```
---

## **[L-05]** - Add a Check for matching `from` and `to` Addresses in `transferBetweenDelegators`

## Summary
The `transferBetweenDelegators` function is used to transfer ENS token delegations, but it lacks a check to ensure that the `from` and `to` addresses are different. Currently, the code executes without any issues even if both addresses are the same, resulting in unnecessary gas consumption to update the same variables and mint and burn ERC1155 tokens for no functional reason. This can lead to a suboptimal user experience.


## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L163-L171

## Tools Used
Manual Review

## Remediation Steps
Add a check and revert the transaction if the `from` and `to` addresses are the same.
```
require(from != to, "From and to addresses must be different");
```

---

## **[L-06]** - use `abi.encode` instead of `abi.encodePacked`

## Summary
The current code employs `abi.encodePacked` for generating hashes by passing different types of parameters. While this approach works, it presents a potential issue due to differences in packed encoding for different types. It's less likely that this will be a critical problem, but using `abi.encode` instead of `abi.encodePacked` is a more robust approach to ensure consistent behavior across different parameter types. This change improves code reliability and consistency.

## Proof of Concept
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L198-L215

## Tools Used
Manual Review

## Remediation Steps
Use `abi.encode` instead of `abi.encodePacked`