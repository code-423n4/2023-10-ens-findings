GAS OPTIMIZATION

In `_delegateMulti()`,  the function call to `Math.max(sourcesLength, targetsLength)` can be cached to save gas in `require()` and `loop` statements.
`Math.min(sourcesLength, targetsLength)` can also be cached in same function.