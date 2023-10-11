# Findings Summary

| ID     | Title                                      | Severity |
| ------ | ------------------------------------------ | -------- |
| [L-01] | ERC20 transfer return value is not checked | Low      |

# Detailed Findings

# [L-01] ERC20 transfer return value is not checked

## Description

The sponsor said on discord: The audit should consider any ERC20-compliant token that implements ERC20Votes as in-scope.    
For the EIP20 standard, transfer failure does not revert transaction, but requires a bool value to be returned.     
The contract does not check the return value. If the token transfer fails, it will succeed silently, resulting in a loss of funds.

## Recommendations

Check transfer return value
