1.require string longer than 32 bytes cost extra gas.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79#L82
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74#L77

2.cache the value of `Math.max(sourcesLength, targetsLength)` instead of calculate mutiple time in loop
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65#L116

3.use `unchecked{++i}` instead of `i++` save more gas
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L88