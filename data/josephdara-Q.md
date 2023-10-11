## No method to get delegate proxy address publicly

The contract in scope ``ERC20MultiDelegate()`` is used to deploy ``ERC20ProxyDelegator()`` for users to delegate their ENS tokens.
This is done internally using:
```solidity
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
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```
This is done after the address is gotten using ``retrieveProxyContractAddress()`` and the address is checked if it does contain code at all. 
However this function is internal, therefore if votes are to be carried out in future there is no public function in this contract or onchain to get a user's proxy addresses from their actual addresses, although though the proxy address can be computed in realtime, the function is not public there exist no mapping to store the address already deployed. 

```solidity
    function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode, 
            abi.encode(_token, _delegate)
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
```
This does introduce complexities for future use of the contract.


I suggest that the visibility of this ``retrieveProxyContractAddress()`` be set to ``public`` to enable easy compatibility and easy usage of the contract to fetch the proxy address for users