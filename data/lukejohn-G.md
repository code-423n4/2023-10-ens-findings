G1. Math.max(sourcesLength, targetsLength) should only be evaluated once. The second time we can use ``amountsLength`` since we have already checked they are equal earlier. 

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

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < amountsLength;

            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }

```

G2. Math.min(sourcesLength, targetsLength) should be evaluated only once, not each time in the iteration.
```diff
 function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

+       uint minLen = Math.min(sourcesLength, targetsLength);

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < amountsLength;

            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex < minLen)) {

                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }

```

G3. ERC20MultiDelegate._processDelegation() calls transferBetweenDelegators(), which will calculate the proxy address again for target although it has been returned by deployProxyDelegatorIfNeeded(). So we can save gas by implementing the transfer directly here:

```diff
 function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

+       address proxyAddressFrom = retrieveProxyContractAddress(token, from);

-        deployProxyDelegatorIfNeeded(target);
+       address proxyAddressTo = deployProxyDelegatorIfNeeded(target);
       
-       transferBetweenDelegators(source, target, amount);

+       token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
