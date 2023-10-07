## Prefer strict inequalities over non-strict inequalities
It is generally recommended to use strict inequalities (<, >) over non-strict inequalities (<=, >=). This is because the compiler will sometimes change a > b to be !(a < b) to accomplish the non-strict inequality. The EVM does not have an opcode for checking less-than-or-equal to or greater-than-or-equal to.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131
```solidity
        assert(amount <= balance);
```
## Do-While loops are cheaper than for loops
If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times;) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85
```solidity
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
## Prefer very large values for the optimizer
The Solidity optimizer focuses on optimizing two primary aspects:

The deployment cost of a smart contract.

The execution cost of functions within the smart contract.

There’s a trade-off involved in selecting the runs parameter for the optimizer.

Smaller run values prioritize minimizing the deployment cost, resulting in smaller creation code but potentially unoptimized runtime code. While this reduces gas costs during deployment, it may not be as efficient during execution.

Conversely, larger values of the runs parameter prioritize the execution cost. This leads to larger creation code but an optimized runtime code that is more cheaper to execute. While this may not significantly affect deployment gas costs, it can significantly reduce gas costs during execution.

Considering this trade-off, if your contract will be used frequently it is advisable to use a larger value for the optimizer. As this will save up gas costs in a long term.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol
## Heavily used functions should have optimal names
The EVM uses a jump table for function calls, and function selectors with lesser hexadecimal order are sorted first over selectors with higher hex order. In other words, if two function selectors, for example, 0x000071c3 and 0xa0712d68, are present in the same contract, the function with the selector 0x000071c3 will be checked before the one with 0xa0712d68 during contract execution.

Hence, if a function is used frequently, it is essential for it to have an optimal name. This optimization increases its chances of being sorted first, thus saving gas costs from further checks (although if there are more than four functions in the contract, the EVM does a binary search for the jump table instead of a linear search).

This also reduces calldata cost (if the function has leading zeros, as zero bytes cost 4 gas, and non-zero bytes cost 16 gas).
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151
## Use branchless algorithms as a replacement for conditionals and loops
For loops have jumps built into them, so you may want to consider loop unrolling to save gas.

Loops don’t have to be unrolled all the way. For example, you can execute a loop two items at a time and cut the number of jumps in half.

This is a very extreme optimization, but you should be aware that conditional jumps and loops introduce a slightly more expensive opcode.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85
```solidity
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```