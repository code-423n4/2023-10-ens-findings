# Gas Optimizations

##

## [G-] Avoid initializing default values for variables its waste of gas 

When you declare a variable but don't explicitly assign it a value, it gets initialized with a default value. The default value depends on the variable type.

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

- 86:  uint transferIndex = 0;
+ 86:  uint transferIndex;

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L86

##

## [G-] Avoid caching ``getBalanceForDelegate(source)`` when it's used only once

Only need the result of getBalanceForDelegate(source) once in your contract, there's no need to cache it. Caching the result would incur gas costs for storing the value and maintaining it, which might not be worth it if you only use that value once

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

129:  uint256 balance = getBalanceForDelegate(source);
130:
131:        assert(amount <= balance);

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131

##

## [G-] Use 3 indexed rule for events in solidity to save gas

You can optimize your events by using the indexed keyword for up to three parameters in an event. By marking certain event parameters as indexed, you make it more efficient for clients to filter and search for specific events in the Ethereum blockchain, which can save gas when querying logs.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

32: event ProxyDeployed(address indexed delegate, address proxyAddress);
33:    event DelegationProcessed(
        address indexed from,
        address indexed to,
        uint256 amount
    );

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L32-L37





