[G-01] Caching Math.max(sourcesLength, targetLength) will save gas

The return value of Math.max(sourcesLength, targetsLength) is used twice in `_delegateMulti`.

Once in the require statement:

```solidity
 require(
    Math.max(sourcesLength, targetsLength) == amountsLength,
    "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
);
```

and once in the loop condition:

```solidity
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
)
```

We can simply cache this by assigning Math.max(sourcesLength, targetsLength) to a variable at the beginning of the function and then use that variable.