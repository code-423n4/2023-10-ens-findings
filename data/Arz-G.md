
## Summary

All optimizations were benchmarked using the protocol's tests, i.e. using the following config: solc version: 0.8.7, optimizer disabled and yarn test test/delegatemulti

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | The target proxy address is calculated twice when _processDelegation() is called | 1 |  2727 |
| [G&#x2011;02] | The max length of source and targets arrays can be cached and used in both require statements | 1 |  319 |
| [G&#x2011;03] | balanceOf() can be used instead of calling ERC1155(this) | 1 |  938 |
| [G&#x2011;04] | The token address can be hardcoded in retrieveProxyContractAddress() instead of receiving it as a param | 1 |  20 |
| [G&#x2011;05] | delegateMulti() and _delegateMulti() can be one external function  | 1 | 54  |
| [G&#x2011;06] | Consider using Solmate instead of OZ | 1 |  815 + 20% cheaper deployment gas |

Total: 6 instances over 6 issues 
Total gas saved: 4873


## Gas Optimizations

### [G&#x2011;01]  The target proxy address is calculated twice when _processDelegation() is called

When `_processDelegation()` is used `deployProxyDelegatorIfNeeded()` is called before `transferBetweenDelegators()` is and in both functions the target proxy is calculated by `retrieveProxyContractAddress()`. 
Because this wastes gas we should only calculate the proxy target address once by calling `deployProxyDelegatorIfNeeded()` which will return the address and then we will pass this address to `transferBetweenDelegators()` to avoid retrieving it again in `transferBetweenDelegators()`


*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 2727 gas*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  517759 |  


*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L133-L134


```solidity

133:   deployProxyDelegatorIfNeeded(target);
134:   transferBetweenDelegators(source, target, amount);

163: function transferBetweenDelegators(
164:     address from,
165:     address to,
166:     uint256 amount
167: ) internal {
168:     address proxyAddressFrom = retrieveProxyContractAddress(token, from);
169:     address proxyAddressTo = retrieveProxyContractAddress(token, to);

```

As you can see here the target address is retrieved twice, first when calling `deployProxyDelegatorIfNeeded()` and then in `transferBetweenDelegators()`

*Should be replaced with:*

```solidity

address proxyTarget = deployProxyDelegatorIfNeeded(target);
transferBetweenDelegators(source, proxyTarget, amount);

```

### [G&#x2011;02]  The max length of source and targets arrays can be cached and used in both require statements

The max length of source and targets arrays can be cached and it can then be used in both require statements and in the loop. This will save gas by calculating the max only once and then caching it. 



*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 319 gas*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  520167 |  



*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L74-L87

```solidity

74: require(
75:     sourcesLength > 0 || targetsLength > 0,
76:     "Delegate: You should provide at least one source or one target delegate"
77: );
78: 
79: require(
80:     Math.max(sourcesLength, targetsLength) == amountsLength,
81:     "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82: );
83: 
84: // Iterate until all source and target delegates have been processed.
85: for (
86:     uint transferIndex = 0;
87:     transferIndex < Math.max(sourcesLength, targetsLength);

```

As you can see here the max is calculated 2 times and we can also use it in the first require statement.

*Should be replaced with:*

```solidity

uint256 maxLength = Math.max(sourcesLength, targetsLength);
require(
    maxLength > 0,
    "Delegate: You should provide at least one source or one target delegate"
);
require(
    maxLength == amountsLength,
    "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
);
// Iterate until all source and target delegates have been processed.
for (
    uint transferIndex = 0;
    transferIndex < maxLength;
    transferIndex++
) {

```



### [G&#x2011;03] balanceOf() can be used instead of calling ERC1155(this)


In `_processDelegation()` we are calling `getBalanceForDelegate()` which calls `ERC1155(this).balanceOf()` but this only wastes gas and can be simplified that `balanceOf()` is called in the assert statement. 



*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 938 gas*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  519548 |  


*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L195

```solidity
195:   return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```

*Should be replaced with:*

```solidity

131: assert(amount <= balanceOf(msg.sender, uint256(uint160(source))));

```


### [G&#x2011;04]  The token address can be hardcoded in retrieveProxyContractAddress() instead of receiving it as a param


`retrieveProxyContractAddress()` is used to calculated the address of the proxy, however everytime we are calling this function we also need to pass the token address which is redundant because we can just hardcode it in the function. 


*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 20 gas*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  520466 |  



*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L199


```solidity

198: function retrieveProxyContractAddress(
199:     ERC20Votes _token,

```

*Should be replaced with:*

```solidity

function retrieveProxyContractAddress(
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode, 
            abi.encode(token, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }
}  

```

### [G&#x2011;05] delegateMulti() and _delegateMulti() can be one external function

`delegateMulti()` calls an internal `_delegateMulti()` but this only wastes gas and it can just be one external function. 


*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 54 gas*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  520432 |  



*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63

```solidity

57: function delegateMulti(
58:     uint256[] calldata sources,
59:     uint256[] calldata targets,
60:     uint256[] calldata amounts
61: ) external {
62:     _delegateMulti(sources, targets, amounts);
63: }

```

*Should be replaced with:*

```solidity

function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        uint256 sourcesLength = sources.length;
        ...

```

### [G&#x2011;06] Consider using Solmate instead of OZ

Solmate library is more optimized than OZ saving gas to the user.


*Gas Savings for delegateMulti() obtained via protocol's tests: Avg 815 gas and 20% cheaper deployment gas costs*

|        |   AVG  |   
| ------ | ------ | 
| Before |  520486 |  
| After  |  519671 |  




*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L7

```solidity
7: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
```
