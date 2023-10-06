Replace strict greater-than-zero operation (> 0) with does-not-equal-zero (!= 0) operation

When checking whether a value is equal to zero, using the construction var != 0 is costs less gas than using var > 0. Note that this is true only when the comparison occurs in a conditional context and the Solidity compiler is using the Optimize

https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L75C1

            sourcesLength > 0 || targetsLength > 0,
           