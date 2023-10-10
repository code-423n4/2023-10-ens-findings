### Gas Optimizations

|ID|Issue|Gas|
|:--:|:---|:--:|
| [[G&#x2011;01]](#g01-rewrite-_delegatemulti-function) | Rewrite `_delegateMulti()` function | 804 |
| [[G&#x2011;02]](#g02-remove-amount-checking-from-_processdelegation) | Remove `amount` checking from `_processDelegation()` | 1140 |

### [G&#x2011;01] Rewrite `_delegateMulti()` function
[`delegateMulti()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57-L63) can be called by a user for following purposes:
- deposit token and delegate vote to designated account
- move delegated vote from one account to another account
- cancel delegation and withdraw token

Therefore it will be more readable if the loop codes are divided into 3 parts:
```diff
    function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
+       uint minLength = Math.min(sourcesLength, targetsLength);
+       uint maxLength = Math.max(sourcesLength, targetsLength);
+       require(
+           maxLength == amountsLength,
+           "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
+       );
+       // Process the delegation transfer between the current source and target delegate pair.
+       for (uint transferIndex = 0; transferIndex < minLength; transferIndex++) {
+           _processDelegation(address(uint160(sources[transferIndex])), address(uint160(targets[transferIndex])), amounts[transferIndex]);
+       }
+       if (maxLength == sourcesLength) {
+           // Handle any remaining source amounts after the transfer process.
+           for (uint transferIndex = minLength; transferIndex < maxLength; transferIndex++) {
+               _reimburse(address(uint160(sources[transferIndex])), amounts[transferIndex]);
+           }
+       } else {
+           // Handle any remaining target amounts after the transfer process.
+           for (uint transferIndex = minLength; transferIndex < maxLength; transferIndex++) {
+               createProxyDelegatorAndTransfer(address(uint160(targets[transferIndex])), amounts[transferIndex]);
+           }
+       }
-       require(
-           Math.max(sourcesLength, targetsLength) == amountsLength,
-           "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
-       );

-       // Iterate until all source and target delegates have been processed.
-       for (
-           uint transferIndex = 0;
-           transferIndex < Math.max(sourcesLength, targetsLength);
-           transferIndex++
-       ) {
-           address source = transferIndex < sourcesLength
-               ? address(uint160(sources[transferIndex]))
-               : address(0);
-           address target = transferIndex < targetsLength
-               ? address(uint160(targets[transferIndex]))
-               : address(0);
-           uint256 amount = amounts[transferIndex];

-           if (transferIndex < Math.min(sourcesLength, targetsLength)) {
-               // Process the delegation transfer between the current source and target delegate pair.
-               _processDelegation(source, target, amount);
-           } else if (transferIndex < sourcesLength) { 
-               // Handle any remaining source amounts after the transfer process.
-               _reimburse(source, amount);
-           } else if (transferIndex < targetsLength) {
-               // Handle any remaining target amounts after the transfer process.
-               createProxyDelegatorAndTransfer(target, amount);
-           }
-       }

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);//@audit-info burn amounts of id(targets) from caller
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");//@audit-info mint amounts of id(targets) to caller
        }
    }
```

### [G&#x2011;02] Remove `amount` checking from `_processDelegation()`
When [`delegateMulti()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57-L63) is called, `amounts` will be checked in [`_burnBatch()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L111), therefore there is no need to check again in `_processDelegation()`:
```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
-       uint256 balance = getBalanceForDelegate(source);

-       assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }

-   function getBalanceForDelegate(
-       address delegate
-   ) internal view returns (uint256) {
-       return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
-   }
```