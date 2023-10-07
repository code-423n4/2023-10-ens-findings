[L‑01] Missing checks for 0 address and 0 amount when setting input arrays

There are 3 instances:

[ERC20MultiDelegate.sol (#L57-L60):](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L60)
```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
```