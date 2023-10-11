1) Can save gas by using  `transferIndex < amountsLength` instead of  `transferIndex < Math.max(sourcesLength, targetsLength)`
Link to code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87

Since `Math.max(sourcesLength, targetsLength)` is already computed, there is no need to compute it each time for the loop iterations. The result is already cached and ready to use.


2) increment ++i costs less than post increment i++

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L88

Increased Costs with Post-Increment: The operation i++ (post-increment) consumes more gas than ++i (pre-increment). The post-increment operation involves incrementing the variable i but returns its initial value, necessitating a temporary variable. This process results in extra gas consumption, approximately an additional 5 gas per iteration.

