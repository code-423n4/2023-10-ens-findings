### NC-1 Inconsistent use of _ (underscore) for function arguments/parameters.

There are functions that make use of _ for function arguments and other functions do not use this 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L44
```
  constructor(
        ERC20Votes _token,
        string memory _metadata_uri
```
Constructor arguments use _underscore "_token, _uri" 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L198
```
 function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
```
Above function makes use of _ underscore on params/arguments

However the rest of the functions do not follow this format 


Impact:--> The above leads to inconsistencies, affects code readability, maintanability

Recommendation: --> It is recommended to make use of of consistency in applying _ to arguments and apply to all functions 

### NC-2 Grammar, and readability of comments

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L23
```
* @dev A utility contract to let delegators to pick multiple delegate

vs 

* @dev A utility contract to allow delegators to pick multiple delegates or delegates
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L52
```
* @dev Executes the delegation transfer process for multiple source and target delegates.

vs

* @dev Executes the delegation transfer process for multiple sources and target delegates.
```


### NC-3  Require strings can be saved in variables (only if not choosing to use Custom Errors) 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L76

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L81

Require strings can be placed in variables and variable used in place of string
This can help prevent errors in copy pasting or typing some of these require strings that are long 
e.g 
```
string provideSource = "Delegate: You should provide at least one source or one target delegate";
require(
            sourcesLength > 0 || targetsLength > 0,
            provideSource
        );
```


### NC-4  Missing events

The following functions would do well to emit events that can be critical for off chain tooling, reporting, monitoring, front ends and more 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57
Emit event when you delegateMulti(....) 


https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L173
Emit event when deployment of Proxy Delegator is done

### NC-5  Use latest versions of OpenZeppelin contracts

Contracts are importing and making use of versions as seen in packages file 
```
"@openzeppelin/contracts": "^4.3.1",
```
Whereas the latest versions of OpenZeppelin contracts are available like OpenZeppelin Contracts 5.0 

