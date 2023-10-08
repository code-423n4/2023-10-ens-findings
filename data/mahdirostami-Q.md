# Title
Use require instead of assert
* * *
- https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131
* * *
## Summary
Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input.

## Impact

The assert function creates an error of type Panic(uint256).

## Tools Used
Manual review
## Recommendations
Use require instead of assert:
```diff
-        assert(amount <= balance);
+        require(amount <= balance, "not enough balance");
```