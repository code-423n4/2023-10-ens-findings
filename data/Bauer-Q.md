## Mismatched Data Type: Passing 0 as a Salt Parameter in Contract Deployment

When creating a new contract instance using the new keyword, the salt parameter should be of type bytes32. However, in the program, it is passed as 0. This discrepancy may lead to unexpected behavior or errors in the contract deployment process, as the salt value is expected to be a unique identifier that influences the resulting contract address.
https://docs.soliditylang.org/en/develop/control-structures.html#salted-contract-creations-create2
```solidity
    new ERC20ProxyDelegator{salt: 0}(token, delegate);
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );    

```
## Recommended Mitigation Steps
It is recommend to use bytes32(0).