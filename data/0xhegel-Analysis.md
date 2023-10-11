
## overview

This project uses a creative way to manage delegation data which leverages  contract creating and ERC1155 .

I highlighted some few special places for consideration.


## 1. Transfer ERC1155 will transfer the ownership of ERC20(ENS)

For example , suppose users A, B ,C. They do the following steps:

1.  A delegate 100 ENS to B by ERC20MultiDelegate contract
```
 // delegateMulti([],[B],[100])

 - New contract(ERC20ProxyDelegator) will be deployed( I personally call it a delegation pool for B)
 - A hold 100 ERC1155
```
   
2. A transfer 100 ERC1155 to C
```
 delegation data is not affected
```
3. C now can now withdraw those ENS by ERC20MultiDelegate
```
 // delegateMulti([B],[],[100])
 C will have 100 ENS
 
```

This is a feature that owner of ERC1155 will own the ability to re-delegate and  own the token(ENS) too .
But this is different from the default delegation feature of ERC20Votes, delegation won't affect the ownership of Token
The end-user should know this by docs etc.

## 2. We may need a simple API to delegate/undelegate to/from a single account

Case: 
I have 100 ens and I want delegate just 50 to some account easily, 
Especially I want to do it in etherscan UI which means I don't want to depend on official UI.
I am not a developer and don't know code.

Solution:
1. Transfer ENS to ERC20ProxyDelegator address  directly.
```
 will add vote balance to related account but the token will be locked.
 we have no way to withdraw. So this no the way.

 Maybe that's why retrieveProxyContractAddress is not a public method. 
 Knowing the address is not useful an dangerous.
```

    
 2. How about add extra functions like this:
```solidity

- delegage(address account, uint256 amount)
- unDelegage(address account, uint256 amount)
(like deposit/withdraw)

The params is easy to input in etherscan ui, so it is user friendly.


```
 

## 3. Possible  gas optimization

I found in the  method ERC20MultiDelegate. _delegateMulti, there is a loop
```
for (uint transferIndex = 0;transferIndex < Math.max(sourcesLength, targetsLength);transferIndex++){

	// ...
	uint256 amount = amounts[transferIndex];
	// if(amount==0) continue; // skip when amount==0? 
	// ...
}
```
When the amount is zero, delegation data will not be effected.
But some logic still need to be executed, especially it include contract creation, when  targetsLength> sourcesLength, and related contract is not yet initialized.
But from a different perspective 'amount' is the user's input, user should know what they do. So it's just a consideration.








### Time spent:
20 hours