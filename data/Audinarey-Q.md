## MISSING CHECK FOR APPROVED DELEGATORS
The sponsor states in one of the main invariants of the [docs page](https://code4rena.com/contests/2023-10-ens#top) that _Tokens should only be transferred between approved delegators_ but in the [```delegateMulti(...)```](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57) function, no check is performed to ensure the ```target``` delegators are approved

## SUGGESTION
Consider implementing a logic in the ```delegateMulti(...)``` function to ensure the ```targets``` are approved before executing delegation transfers.