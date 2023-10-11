# Gas Optimizations

## [G-1] ``ERC20MultiDelegate.getBalanceForDelegate()`` could be optimized to reduce gas costs

### Saves ``5000 GAS``

If the delegate value is the zero address, then the expression uint256(uint160(address(0))) will return the value 0. This is because the zero address is not a valid Ethereum address and it is used to represent a lack of ownership or a default value.

The ERC1155.balanceOf() function will always return 0 if the id value is 0. This is because the ERC1155 standard does not allow tokens with the ID of 0 to be minted or transferred.

```
function balanceOf(address account, uint256 id) public view virtual returns (uint256) {
        return _balances[id][account];
    }

```

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

function getBalanceForDelegate(address delegate) internal view returns (uint256) {
+    if (delegate == address(0)) {
+        return 0;
+    }
    return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
}

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L192-L196

##

## [G-2] ``require`` is more gas efficient than ``assert ``

Approximate breakdown of the gas costs for each statement
- require statement: 14 gas
- assert statement: 294 gas

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

131:  assert(amount <= balance);

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131

##

## [G-3] Unnecessary ``computation`` in for ``loop``

### Saves ``294 GAS`` for each Iterations 

Computing Math.max(sourcesLength, targetsLength) within the loop condition during each iteration is unnecessary and less efficient. It's better to calculate this value once and store it in a variable before the loop to avoid redundant computations.

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

+ uint256 maxIterations = Math.max(sourcesLength, targetsLength) ;

 for (
            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < maxIterations ;
            transferIndex++
        ) {

```


##

## [G-4] Redundant checks of ``(sourcesLength > 0) `` and ``(targetsLength > 0)``

The values already checked in ``require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        ); `` require block . So checking again is redundant . This costumes extra ``200 Gas`` for every iterations. 

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

- 110: if (sourcesLength > 0) {
111:            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
- 112:        }
- 113:        if (targetsLength > 0) {
114:            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
- 115:        }

```
##

## [G-5] Avoid caching functions and variables when they are only used once to save gas

Only need the result of getBalanceForDelegate(source) once in your contract, there's no need to cache it. Caching the result would incur gas costs for storing the value and maintaining it, which might not be worth it if you only use that value once

### Saves ``25 GAS``

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

- 129:  uint256 balance = getBalanceForDelegate(source);
130:
- 131:        assert(amount <= balance);
+ 131:        assert(amount <= getBalanceForDelegate(source));

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131

### Saves ``265 GAS``

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
-        bytes memory bytecode = abi.encodePacked(
-            type(ERC20ProxyDelegator).creationCode, 
-            abi.encode(_token, _delegate)
-        );
-        bytes32 hash = keccak256(
-            abi.encodePacked(
-                bytes1(0xff),
-                address(this),
-                uint256(0), // salt
-                keccak256(bytecode)
-            )
-        );
+        return address(uint160(uint256( keccak256(
+            abi.encodePacked(
+                bytes1(0xff),
+                address(this),
+                uint256(0), // salt
+                keccak256(abi.encodePacked(
+            type(ERC20ProxyDelegator).creationCode, 
+            abi.encode(_token, _delegate)
+        ))
+            )
+        ))));
+    }

```

### Saves ``200 GAS ``

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
-        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-        address proxyAddressTo = retrieveProxyContractAddress(token, to);
-        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
+        token.transferFrom(retrieveProxyContractAddress(token, from), retrieveProxyContractAddress(token, to), amount);
    }

```

### Saves ``200 GAS``

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
-        address proxyAddress = deployProxyDelegatorIfNeeded(target);
-        token.transferFrom(msg.sender, proxyAddress, amount);
+        token.transferFrom(msg.sender, deployProxyDelegatorIfNeeded(target), amount);
    }

function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
-        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
-        token.transferFrom(proxyAddressFrom, msg.sender, amount);
+        token.transferFrom(retrieveProxyContractAddress(token, source), msg.sender, amount);
    }

```

### ``20 GAS``

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

  uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
-        uint256 amountsLength = amounts.length;

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
-            Math.max(sourcesLength, targetsLength) == amountsLength,
+            Math.max(sourcesLength, targetsLength) == amounts.length,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );


```

##

## [G-6] Avoid initializing default values for variables its waste of gas 

When you declare a variable but don't explicitly assign it a value, it gets initialized with a default value. The default value depends on the variable type.

```diff
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

- 86:  uint transferIndex = 0;
+ 86:  uint transferIndex;

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L86


##

## [G-7] Use 3 indexed rule for events in solidity to save gas

You can optimize your events by using the indexed keyword for up to three parameters in an event. By marking certain event parameters as indexed, you make it more efficient for clients to filter and search for specific events in the Ethereum blockchain, which can save gas when querying logs.

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

32: event ProxyDeployed(address indexed delegate, address proxyAddress);
33:    event DelegationProcessed(
        address indexed from,
        address indexed to,
        uint256 amount
    );

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L32-L37

##

## [G-8] ``_delegateMulti()``can be more gas optimized 

If any of these ``transferIndex < Math.min(sourcesLength, targetsLength)``,``(transferIndex < sourcesLength)``,``(transferIndex < targetsLength)`` conditions evaluate to false, it means that there are no more items to process in the respective arrays (``sources`` or ``targets``), and continuing the loop would indeed waste gas. if these checks ``false`` .

```solidity
FILE: 2023-10-ens/contracts/ERC20MultiDelegate.sol

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






