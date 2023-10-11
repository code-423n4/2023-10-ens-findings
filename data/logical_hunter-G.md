# Use constant instead of type(uint256).max
- https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L17
- Instead of using type(uint256).max use a constant to specify the maximum value so that the value doesn’t need to be calculated each time the child contract’s constructor is called.

# >= is cheaper than >
- https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L75
- Use >= 1 instead of > 0
- >= uses lesser gas than >

# <= is cheaper than <
- https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98
- Use <= instead of <
