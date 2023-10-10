## An external call is used instead of an internal call

### Lines of code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L195

### Description

The `balanceOf` function is called as `ERC1155(this).balanceOf(...)`. This will initiate a new EVM `CALL` to the same contract. On the other hand, it is possible to call this function by `balanceOf(...)`, which will result in an internal `jump`. Since both of these ways will result in the same answer, it is better just to use the internal call in order to save gas (on additional `mstore`, `call`, `mload`).

### Recommended Mitigation Steps

Replace `ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)))` with `balanceOf(msg.sender, uint256(uint160(delegate)))`

## Recalculation of the Math.max()

### Lines of code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

### Description

The expression `Math.max(sourcesLength, targetsLength)` is recalculated inside the for-loop.

### Recommended Mitigation Steps

Consider using `amountsLength`, since it is [checked](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79-L82) to be equal to `Math.max(sourcesLength, targetsLength)`

## Recalculation of the Math.min()

### Lines of code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98

### Description

The expression `Math.min(sourcesLength, targetsLength)` is recalculated inside the for-loop on each iteration.

### Recommended Mitigation Steps

Consider declaring an additional variable outside of the for-loop and using it.
