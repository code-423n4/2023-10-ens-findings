### Low risk issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | The user will fail to reimburse from his proxy after processing delegation to it  | 1 | 
| [L&#x2011;02] | The uri is not unique for each token  | 1 |  
| [L&#x2011;03] | Events are not emitted when depositing or withdrawing  | 1 | 
| [L&#x2011;04] | The user can accidentally delegate to a zero address  | 2 |

Total: 5 instances over 4 issues 

## [L&#x2011;01] The user will fail to reimburse from his proxy after processing delegation to it


If a user wants to call `delegateMulti()` where he first processes delegation transferring tokens to his own proxy and then he wanted to reimburse from it(in the same tx) then the tx will revert because the ERC1155 tokens are burned before they are minted.

Example:

1. Alice delegates 100 $ENS to Bob
2. Later she decides that she wants to transfer 50 ENS to her own proxy and withdraw(reimburse) 25 $ENS.
3. Alice calls `delegateMulti()` where the first loop iteration will call `_processDelegation()` and transfer the 50 $ENS to her proxy
4. The second iteration will then reimburse 25 $ENS from her proxy
5. The problem is that this tx will fail because the ERC1155 tokens are burned before they are minted, those 50 ERC1155 tokens are minted after the 25 ERC1155 tokens burned which Alice doesnt have and the tx reverts. 


### Impact

If the user wanted to reimburse after the delegation then the tx would revert. 


### Code Snippet

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98-L115

```solidity

98: if (transferIndex < Math.min(sourcesLength, targetsLength)) {
99:     // Process the delegation transfer between the current source and target delegate pair.
100:     _processDelegation(source, target, amount);
101: } else if (transferIndex < sourcesLength) {
102:     // Handle any remaining source amounts after the transfer process.
103:     _reimburse(source, amount);
104: } else if (transferIndex < targetsLength) {
105:     // Handle any remaining target amounts after the transfer process.
106:     createProxyDelegatorAndTransfer(target, amount);
107: }
108: }
109
110: if (sourcesLength > 0) {
111:     _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
112: }
113: if (targetsLength > 0) {
114:     _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
115: }

```

As you can see here, the tokens are first burned before they are minted so if someone wanted reimburse after the delegation then they would burn tokens that they didnt receive yet



### Recommended Mitigation Steps

The solution here is a problem because minting before burning is not ideal and can introduce reentrancy attacks so its up to the sponsor if they want to fix this or no. 


## [L&#x2011;02] The uri is not unique for each token

The `uri()` function here is not overriden an it just returns the string which can be a problem because someone might want to have a uri unique for each token. 


### Impact

Although ENS might not need this other people might so the uri returned wont be unique for each token and the metadata wont work. 


### Code Snippet

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L47

The uri here is set in the constructor or it can also be set by the owner however the uri() function that is in the ERC1155.sol is not overriden and it will just return the uri without the token id. 



### Recommended Mitigation Steps

Consider adding a bool in the constructor, if this bool is true then the uri() function will return the uri + token id by abi.encodePacking, if the bool is false then it would just return the string as it is right now. 



## [L&#x2011;03] Events are not emitted when depositing or withdrawing


When `_processDelegation()` is called DelegationProcessed event is emitted however this event is not emitted when reimbursing or depositing in `_reimburse()` and `createProxyDelegatorAndTransfer()`. 


### Impact

This can lead to confusion and no data being captured by event listeners. 

### Code Snippet

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155


As you can see DelegationProcessed is only emitted in `_processDelegation()` but not in `_reimburse()` and `createProxyDelegatorAndTransfer()`.

### Recommended Mitigation Steps

Emit the events in both of the functions.



## [L&#x2011;04] The user can accidentally delegate to a zero address

Because there are no zero address checks in `delegateMulti()` the user can accidentally delegate to a zero address and he might not know about it. 


### Impact

The user can accidentally delegate to a zero address wihtout knowing about it because the tx wont revert thinking he delegated to the right address. 

### Code Snippet

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98-L107

As you can see `_processDelegation()` and `createProxyDelegatorAndTransfer()` can be called with target being a zero address.

### Recommended Mitigation Steps

Consider checking if target != address(0) before calling `_processDelegation()` or `createProxyDelegatorAndTransfer()`


