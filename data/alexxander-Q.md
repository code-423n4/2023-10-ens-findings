## The for-loop in `_delegateMulti(...)` should skip iterations when `amount = 0`
Applications integrating with the Multi Delegate contract or a user mistake could make so that some of the input `amounts[ ]` entries contain a 0 amount. Skipping an iteration in `_delegateMutli(...)` on a 0 amount could save gas and also prevent Out Of Gas errors.
#### Links
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L116
## User cannot transfer tokens from Proxy A to Proxy B to Proxy C in a single transaction
This issue could make some weird integrations with Multi Delegate impossible. This is caused because at the end of the for loop in `_delegateMulti(...)` we first burn tokens for the corresponding sources and then mint the new tokens. If we transfer from Proxy A to Proxy B and then to Proxy C (in a single `_delegateMulti(...)` call) burning from Proxy B occurs before minting to Proxy B which will cause a revert. This is more of a Informational issue towards the users, the current implementation of burning and minting seems very secure and if it is changed it might introduce a possibility for a reentrancy attack through the `onERC1155Received` hook.
#### Links
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L110-L115
## Unnecessary `retrieveProxyContractAddress(...)` invocation in `_processDelegation(...)`
In `_processDelegation(...)` we first check if there is an existing proxy for the `target` address through the function `deployProxyDelegatorIfNeeded(...)` and this function returns the proxy address, however it is not utilized. After that `transferBetweenDelegators(...)` is invoked for the source and target and for both addresses we invoke `retrieveProxyContractAddress(...)`. The second invocation for the target address retrieval could be replaced by a parameter that is set to the output of  `deployProxyDelegatorIfNeeded(...) ` that was called beforehand. 
#### Links
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124-L137
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L173-L189
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L163-L170
## Unused `_metadata_uri` in the constructor
Currently the  `_metadata_uri` parameter in the constructor is not used. Either remove it or set the uri to the supplied parameter.
#### Links
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L44-L49