## Gas Optimizations

| Number | Issue | Instances | Estimated Gas Saved |
| --- | --- | --- | --- |
| G-01 | Use constant instead of type(uint256).max | 1 | 24 |

# [G-01] Use constant instead of `type(uintx).max`

Total Instances: `1` 

Can save gas by using a constant variable with the same value as the expression, as constant variables are stored in the contract's bytecode.

Each of the recommendations below saves 24 gas (Tested in Remix)

```solidity
uint private constant MAX_UINT256_NUMBER = 115792089237316195423570985008687907853269984665640564039457584007913129639935;
uint private constant MAX_UINT256_HEX = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
uint private constant MAX_UINT256_EXPONENTIATION = 2**256 - 1;
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L17