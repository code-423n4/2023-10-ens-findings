## L-01 @Dev of Natspec of ERC20MultiDelegate is gramatically wrong
At https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L23 it says " A utility contract to let delegators to pick multiple delegate" which should be corrected to " A utility contract to let delegators pick multiple delegates"

## L-02 Math.max(sourcesLength, targetsLength) could be computed once and reused
It is computed here https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80 and a few lines later again here https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87. Write it into a local variable and reuse it. Saves gas for computation and makes the for loop way easier to read (may not even break into multiple lines anymore).

## L-03 Could use uint160 directly for sources and targets parameters in delegateMulti() interface
Since sources and targets will be casted down to uint160 (https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L91-L95) and then to address the uint160 datatype could be used directly. This would avoid casts and would als prevent the case where uint256 values passed within sources or targets result in the same address due to the cast.

## L-04 Local variables passed to transferFrom function should use consistent naming

There is 3 occasions where the transferFrom function is called:
1) https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L147-L148C15
2) https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L159-L160
3) https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L168-L170

The naming of the from and to variables is inconsistent.

A consistent pattern to follow would be to always add "From" suffix to all variables representing the **from** address of `transferFrom()` calls and a "To" suffix to all variables representing the **to** address of `transferFrom()` calls.

## L-05 Test should use newDelegates variables instead of delegates
The check is iterating the newDelegates array but uses the delegates array: https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L418-L423. The test only passes since delegates and newDelegates have the same size. The tests should be inspected for more of these (copy-paste) errors to increases test robustness.

