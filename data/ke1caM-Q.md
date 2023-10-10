[L-01] Creating proxy with create2 does not follow best practices regarding address check.

## Summary

Precalculated address should be the same as the address created with create2. There are no checks that validate correctness of proxy's address.

## Relevant links

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L173-L189

## Vulnerability details

In `deployProxyDelegatorIfNeeded` we precalculate proxy address with `retrieveProxyContractAddress(token, delegate)` and store it in `address proxyAddress`. If the code at this address exists we return the calcualted address. However when the address does not have associated code the function deploys new `ERC20ProxyDelegator` proxy and returns precalculated address. `deployProxyDelegatorIfNeeded` does not check if the created address is the same as precalculated one.

In Solidity documentation we can see that precalculated address is checked with the newly created proxy address which ensures that the proxy creation was completed correctly.

Code from Solidity documentation in regard of create2:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
contract D {
    uint public x;
    constructor(uint a) {
        x = a;
    }
}

contract C {
    function createDSalted(bytes32 salt, uint arg) public {
        // This complicated expression just tells you how the address
        // can be pre-computed. It is just there for illustration.
        // You actually only need ``new D{salt: salt}(arg)``.
        address predictedAddress = address(uint160(uint(keccak256(abi.encodePacked(
            bytes1(0xff),
            address(this),
            salt,
            keccak256(abi.encodePacked(
                type(D).creationCode,
                abi.encode(arg)
            ))
        )))));

        D d = new D{salt: salt}(arg);
        require(address(d) == predictedAddress);
    }
}
```

As we can see newly created contract's address is check with precalculated one.

## Recommendations

Implement a check after creating new contract. It might look like this:

```
function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            ERC20ProxyDelegator proxyDelegator = new ERC20ProxyDelegator{salt: 0}(token, delegate);
            require(address(proxyDelegator) == proxyAddress);
            emit ProxyDeployed(delegate, proxyAddress);
        }

        return proxyAddress;
    }
```

[L-02] Delegation of zero votes.

## Summary

In `delegateMulti` it is possible to delegate zero votes. Input check can be implemented to ensure that calling `delegateMulti` in fact does some changes to the current state.

## Details

Zero votes delegation does not modify current state and could be omitted. Not only this operation does not change anything, it also wastes gas. I would be wise to implement a check in for loop that ensures zero values are not present. If so for loop should skip this iteration.

## Recommendations

Check could look like this:

```solidity
    require(amounts[transferIndex] > 0);
```

[L-03] Reentrancy guard not implemented.

## Summary

`nonReentrant` modifier guards against reentrancy attack. This attack could occur when using low level calls, transfering ether and with some ERCs. ERC1155 standard implements an external call after minting or transfering tokens.

In this scenario the code follows checks, effects and interactions pattern which helps to mitigate reentrancy attack. However it is wise to implement such check to ensure that any reentrant call can be made.

Also it would prevent some weird ERC20 with votes extension from malicious actions when working with this code implementation.

## Recommendations

Add `nonReentrant` modifier in `delegateMulti` function.

[L-04] `salt` variable does not follow the best practices.

## Summary

By specifying the `salt` option, which is `bytes32` type, we instruct the contract creation process to use a more secure mechanism for computing the contract address. Setting `salt` to more random value we ensure that address of the newly created contract is more unique meaning more secure.

## Recommendations

Consider changing `salt` value to something like `msg.sender` and constructor parameters.
