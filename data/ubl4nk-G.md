### No need to call `Math.min()` inside loop, cache the result outside loop to save gas
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98
No need to call `Math.min(sourcesLength, targetsLength)` in a for-loop, instead of that you can call this function outside the loop and save the result a variable.
It removes some additional op(byte)-codes and saves gas.
