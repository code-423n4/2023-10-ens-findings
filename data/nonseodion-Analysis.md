## Description
The `ERC20MultiDelegate` contract improves on `ERC20Votes`, an ERC20 extension that supports Compound-like voting and delegation. It allows addresses to delegate their voting power to multiple delegates, unlike `ERC20Votes` which supports only one delegate. 

The key contracts in the protocol are: 
- **ERC20MultiDelegate**: It helps addresses to delegate to multiple delegates and is an ERC1155 token that records much voting power an address has given each delegate. 
- **ERC20ProxyDelegator**: It holds the tokens delegated to a delegate.

## Evaluation Approach
I started studying the code from the ERC20MultiDelegate contract, specifically its `delegateMulti` function. This allowed me to evaluate more than 90% of the code since it is the only external function. I also evaluated parts of the ERC1155 and ERC20Votes contracts that were called by this contract. This took about 4 hours. After evaluation, I considered some attack scenarios before drawing the [mind map](https://drive.google.com/file/d/1gg1nKXyeIofAMknkLKdOdw0VpC5K0k-Z/view?usp=sharing). 

## Mechanisim Review

### Delegation
The ERC20MultiDelegate contract allows a user to delegate, redelegate and undelegate his voting power using just the `delegateMulti` function. It also allows the user to delegate and redelegate to different delegates or undelegate and redelegate to different delegates in one single `delegateMulti` function call.  It doesn't allow the user to perform the three actions in one call though: delegate, undelegate and redelegate. 

### Protecting assets
The developer employs a proxy contract, ERC20ProxyDelegator that holds all the tokens delegated to a delegate. This contract in turn delegates all its tokens to the delegate. So to the ERC20Votes contract the delegate has a lot of tokens delegated to him from the address of the proxy contract. Only the ERC20ProxyDelegator contract has access to transfer these tokens in and out of the proxy contract.

## Systemic Risks
### Loss of all ERC20Vote Balance
Only the ERC20MultiDelegate contract can transfer tokens out of the proxy contracts since it is given infinite allowance by each proxy contract upon deployment. If the contract is somehow compromised to allow a `transferFrom` from any proxy contract all the tokens can be lost.

### Stuck ERC20Vote Balance
As earlier mentioned only the ERC20MultiDelegate contract can transfer tokens out of the proxy contracts. If it can no longer function e.g grief attack or self destruct then each delegator will never be able to retrieve his tokens from the proxy contract.


### Time spent:
15 hours