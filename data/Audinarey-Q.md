## MISSING CHECK FOR APPROVED DELEGATORS
The sponsor states in one of the main invariants of the [docs page](https://code4rena.com/contests/2023-10-ens#top) that _Tokens should only be transferred between approved delegators_ but in the ```delegateMulti(...)``` function, no check is performed to ensure the ```target``` delegators are approved

## SUGGESTION
Consider implementing a logic in the ```delegateMulti(...)``` function to ensure the ```targets``` are approved before executing delegation transfers.