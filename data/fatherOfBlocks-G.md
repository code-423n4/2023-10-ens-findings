**contracts/ERC20MultiDelegate.sol**
- L87 - In the for loop Math.max(sourcesLength, targetsLength) is constantly executed, this operation is executed in each iteration therefore it would be beneficial to do the operation before and put the resulting value in the loop.

