# [G-01] Do not calculate `Math.max(sourcesLength, targetsLength)` on every loop iteration

```
 for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )
```

Instead of calculating `Math.max(sourcesLength, targetsLength)` on every loop iteration, cache that result and use it once, e.g.: 

```
 uint transferIndexEnd = Math.max(sourcesLength, targetsLength);
 for (
            uint transferIndex = 0;
            transferIndex < transferIndexEnd;
            transferIndex++
        )
```

Moreover, please notice, that `Math.max(sourcesLength, targetsLength)` is also being calculated in the require-statement before the loop:
```
require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

This implies, that the result of `Math.max(sourcesLength, targetsLength)` can be cached even before the `require` and then used both inside `require` and inside the loop:

```
  uint256 maxSourceAndTargetsLength = Math.max(sourcesLength, targetsLength);

  require(
            maxSourceAndTargetsLength  == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < maxSourceAndTargetsLength;
            transferIndex++
        )
```

# [G-02] Do not calculate `Math.min(sourcesLength, targetsLength)` on every loop iterations

The length of the `targetsLength` and `sourcesLength` is defined before the loop and does not change during the loop iterations.
This implies, that calculating `Math.min(sourcesLength, targetsLength)` on every loop iteration is a waste of gas:

```
            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
```

`Math.min(sourcesLength, targetsLength)` can be cached into a local variable declared outside the loop, instead of calculating the minimum value on every loop iteration.

# [G-03] Unnecessary declaration of `amountsLength` in `_delegateMulti()`
`amountsLength` is used only once, which implies, that it does not need to be declared at all.
Change:

```
 require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

to:

```
 require(
            Math.max(sourcesLength, targetsLength) == amounts.length,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

and remove line 72: ` uint256 amountsLength = amounts.length;`.

# [G-04] Unnecessary declaration of `balance` in `_processDelegation()`
`balance` is used only once, which implies, that it does not need to be declared at all:

Change:

```
 uint256 balance = getBalanceForDelegate(source);

 assert(amount <= balance);
```

to:

```
assert(amount <= getBalanceForDelegate(source));
```


# [G-05] Unnecessary declaration of `proxyAddressFrom` variable in `_reimburse`

`proxyAddressFrom` is used only once, which implies, that it does not need to be declared at all.

Change:

```
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

to:

```
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        token.transferFrom(retrieveProxyContractAddress(token, source), msg.sender, amount);
    }
```

# [G-06] Unnecessary declaration of `proxyAddress` variable in `createProxyDelegatorAndTransfer`

`proxyAddress` is used only once, which implies, that it does not need to be declared at all.

Change:

```
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
```

to:

```
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        token.transferFrom(msg.sender, deployProxyDelegatorIfNeeded(target), amount);
    }
```

# [G-07] Unnecessary declaration of `proxyAddressFrom`, `proxyAddressTo` variables in `transferBetweenDelegators`

`proxyAddressFrom` and `proxyAddressTo` are used only once, which implies, that it does not need to be declared at all.

Change:

```
    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```

to:

```
    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        token.transferFrom(retrieveProxyContractAddress(token, from), retrieveProxyContractAddress(token, to), amount);
```


# [G-08] `for`-loop in `_delegateMulti` can be optimized

In the current implementation, the loop performs `Math.max(sourcesLength, targetsLength)` iterations:

```
for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )
```

In each of these iterarations, two `trenary operator`-conditions are being executed:

```
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
```

This can be reduced if we'd divide the loop into two parts.

1. First loop should iterate from `0` to `Math.min(sourcesLength, targetsLength)`
2. The second loop should iterate from `Math.min(sourcesLength, targetsLength)` to `Math.max(sourcesLength, targetsLength)`.

Let's consider the first loop. When we looping from `0` to `Math.min(sourcesLength, targetsLength)`, then, we can be sure, that both `transferIndex` is smaller than both `sourcesLength` and `targetsLength`.
In that case, we don't need to perform below check:

```
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
```

since they always evaluate to true, thus above code snippet can be rewritten to:

```
address source = address(uint160(sources[transferIndex]));
address target = address(uint160(targets[transferIndex]))
```

Those `ternary operator`-conditions should be only in the second loop, when we iterate from `Math.min(sourcesLength, targetsLength)` to `Math.max(sourcesLength, targetsLength)`.

Dividing the main `for`-loop into two smaller loops will save gas on `trenary operator`-conditions. We won't need to perform `2 * (Math.max(sourcesLength, targetsLength) - Math.min(sourcesLength, targetsLength) )` `ternary-operator` conditional checks.

The code can look like this:

```
           for (
            uint transferIndex = 0;
            transferIndex < Math.min(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
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

        for (
            uint transferIndex = Math.min(sourcesLength, targetsLength);
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
```

Please take into a consideration, that for the clarity of the code, we did not enforced additional optimizations which were suggested in the previous paragraphs of this report (such as: `transferIndex++` should be `unchecked`, `Math.min` and `Math.max` result should be calculated once and cached).