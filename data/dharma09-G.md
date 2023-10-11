## Gas summary

- [Cache redundant calculation of `Math.max(sourcesLength, targetsLength)` in `_delegateMulti()`](#)
- [State variables should be cached in stake](#)
- [Use assembly for loops to save gas](#)
- [Declare the variables outside the loop](#)
- [Use Solady library for better gas optimization](#)

### Cache redundant calculation of `Math.max(sourcesLength, targetsLength)` in `_delegateMulti()`

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79C7-L89C12

```solidity
File: contracts/ERC20MultiDelegate.sol
65: function _delegateMulti(
66:        uint256[] calldata sources,
67:        uint256[] calldata targets,
68:        uint256[] calldata amounts
69:    ) internal {
...
79:        require(
80:            Math.max(sourcesLength, targetsLength) == amountsLength, //@audit Cache calculation
81:            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82:        );
83:
84:        // Iterate until all source and target delegates have been processed.
85:        for (
86:            uint transferIndex = 0;
87:            transferIndex < Math.max(sourcesLength, targetsLength);
88:            transferIndex++
89:        ) {
...
107:            }
108:        }
```

## State variables should be cached in stake

save gas 100 per instance

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144

```solidity
File: contracts/ERC20MultiDelegate.sol
144: function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source); //@audit cache token
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163

```solidity
File: contracts/ERC20MultiDelegate.sol
163: function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from); //@audit
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173

```solidity
File: contracts/ERC20MultiDelegate.sol
173: function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate); //@audit

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```

## Use assembly for loops to save gas

Saves `350 GAS` for every iteration from `1 instances`

Assembly is more gas efficient for loops. Saves minimum `350 GAS` per iteration as per remix gas checks

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79C7-L89C12

```solidity
File: contracts/ERC20MultiDelegate.sol
85:        for (
86:            uint transferIndex = 0;
87:            transferIndex < Math.max(sourcesLength, targetsLength);
88:            transferIndex++
89:        ) {
```

## Declare the variables outside the loop

Per iterations saves `26 GAS`

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L96

```solidity
File: contracts/ERC20MultiDelegate.sol
96: uint256 amount = amounts[transferIndex]; //@audit
```

### Use Solady library for better gas optimization

use [solady](https://github.com/Vectorized/solady) library

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L6C1-L7C60

```solidity
File: contracts/ERC20MultiDelegate.sol
6: import "@openzeppelin/contracts/access/Ownable.sol"; //@audit
7: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol"; //@audit
```