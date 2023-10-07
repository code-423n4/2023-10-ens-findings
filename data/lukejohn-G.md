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

G3. Checking the proxy address of a given address is really expensive. It is better to introduce a mapping called ``proxyAddresses``, we reimplement the following functions to save gas:

```diff
+  mapping(address => address) proxyAddresses;

   function deployProxyDelegatorIfNeeded(
        address delegate
    ) public returns (address proxyAddress) {
         proxyAddress = proxyAddresses[delegate];

         if(proxy == address(0))  
            proxy = address(new ERC20ProxyDelegator{salt: 0}(token, delegate));
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }

 function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = proxyAddresses[from];
        address proxyAddressTo = proxyAddresses[to];
        require(proxyAddressFrom != address(0) && proxyAddressTo != address(0);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }

function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
-        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
+        address proxyAddressFrom = proxyAddresses[source];

        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

```

G4  To save gas, for function _delegateMulti(), there is no need to calculate ``source`` and ``target``, which is a waste of gas due to comparison of ``transferIndex < sourcesLength`` and ``transferIndex < targetsLength``. These two comparisons can be eliminated. 

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
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
-            address source = transferIndex < sourcesLength
-                ? address(uint160(sources[transferIndex]))
-                : address(0);
-            address target = transferIndex < targetsLength
-                ? address(uint160(targets[transferIndex]))
-                : address(0);
-            uint256 amount = amounts[transferIndex];

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
-                _processDelegation(source, target, amount);
+                _processDelegation(sources[transferIndex], targets[transferIndex], amount); 
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
-                _reimburse(source, amount);
+                _reimburse(sources[transferIndex], amount);

            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
-                createProxyDelegatorAndTransfer(target, amount);
+                createProxyDelegatorAndTransfer(targets[transferIndex], amount);

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

G5. For for function _delegateMulti(), the comparisons of transferIndex < Math.min(sourcesLength, targetsLength, transferIndex < sourcesLength, and transferIndex < targetsLength is a waste of gas since they will be conducted for each iteration. In the following, we differentiate the three cases outside of the loop and thus save gas by not comparing for each iteration. We only need to compare once. in addition, we move the ``_burnBatch`` statement to the beginning to save some gas since when ``_burnBatch`` fail, there is no sufficient balance for the batch transfer, there is no need to execute the rest. 

```diff
   function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        if(sourcesLength == targetsLength){
           for(int i; i < sourcesLength; i++) _processDelegation(sources[i], targets[i], amounts[i]);
        }
        else if(sourcesLength < targetsLength){
           for(int i; i < sourcesLength; i++) _processDelegation(sources[i], targets[i], amounts[i]);
           for(int i = sourcesLength; i < targetsLength; i++) {
                createProxyDelegatorAndTransfer(targets[i], amount);
           } 
       }
       else{
          for(int i; i < targetsLength; i++) _processDelegation(sources[i], targets[i], amounts[i]);
          for(int i = targetsLength; i < sourcesLength; i++) {
                _reimburse(sources[i], amount);
           }          
       }

        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }
```


