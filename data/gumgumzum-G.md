### Math.min and Math.max can be cached in `_delegateMulti`
```solidity
    function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        (uint256 minLength, uint256 maxLength) = sourcesLength < targetsLength
            ? (sourcesLength, targetsLength)
            : (targetsLength, sourcesLength);

        require(
            maxLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            maxLength == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < maxLength;
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

            if (transferIndex < minLength) {
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
### An extra `retrieveProxyContractAddress` can be saved in `_processDelegation` by inlining `transferBetweenDelegators`

```solidity
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);
        
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        address proxyAddressTo = deployProxyDelegatorIfNeeded(target);

        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

        emit DelegationProcessed(source, target, amount);
    }
```