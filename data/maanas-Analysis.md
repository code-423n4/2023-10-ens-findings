## Approach
Since apart from the standard ERC-1155 and Ownable functions, `delegateMulti` is the single external user callable function present in the contract, most of the time was spent analysing this function tracking the state changes. Then other functions related with token transfers, `_mintBatch` and `_burnBatch` of ERC-1155 and `transferFrom` of ERC20Votes was checked for any attack vectors in the context of the current implementation. 
Gas analysis was performed by modifying the code and including tests to compare the previous and newer gas consumption.

## Mechanism Review
The contract attempts to allow delegators to easily delegate and manage their voting power across multiple addresses. To multi-delegate, a proxy contract is deployed for each delegate address. The proxy contract delegates itself to the delegatee address and approves ERC20MultiDelegate to spend all its tokens in its constructor and does nothing else. Hence the vote delegation is managed by transferring user's tokens across these proxy contracts. The accounting of user's delegation is managed by ERC-1155 which represents each delegate as a separate token.

## Codebase analysis
The ERC-1155 and Ownable portions of the contract are inherited from Openzeppelin library which are well tested and used although more gas efficient variants exist.  
In `delegateMulti()` the contract deviates from its actual state as the action on tokens ie. `_burnBatch()` and `_mintBatch()` only occur at the end of the execution. I couldn't find any issue arising due to this as the only dependent action that could be performed in the current implementation is using the tokens from the same source multiple times within a single call. This would be handled by the `_burnBatch` function.  

## Centralization risks
None. The only privileged action is setting the metadata uri of ERC-1155 token. 



### Time spent:
15 hours