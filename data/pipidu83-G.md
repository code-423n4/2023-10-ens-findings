## Use assembly to emit events

[Reference](https://code4rena.com/reports/2023-08-goodentry#g-08-use-assembly-to-emit-events)

```
136        emit DelegationProcessed(source, target, amount);

187        emit ProxyDeployed(delegate, proxyAddress);
```

## Make event parameters indexed to save gas

Using ```indexed``` events will help reduce gas costs. Events are non-indexed by default which means the entire event data is stored which can be expensive in case of large amounts of data.

With ```indexed``` parameters, arguments will be stored in a different data structure easily accessible, meaning the search for specific event occurrences will be overall less costly.

Setting up to 3 event parameters (which is the maximum amount of indexed parameters allowed) to ```indexed``` will hence help save on gas costs.

```
32    event ProxyDeployed(address indexed delegate, address proxyAddress);

33    event DelegationProcessed(
        address indexed from,
        address indexed to,
        uint256 amount
    );
```

## Using > 0 costs more gas than != 0 when used on a uint in a require() statement

```
74        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
```

## Declare variables outside of the loop to save gas

Solidity will create a new instance of the variable for each iteration, which is suboptimal in terms of gas costs, and variables notably array’s length should be defined outside of the loop.

```
87            transferIndex < Math.max(sourcesLength, targetsLength);

98            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
```
## Use custom errors instead of require/assert

It will lead to cheaper deployment cost as well as cheaper runtime cost when the revert condition is met.

```
74        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

79        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

131        assert(amount <= balance);
```

## ++i costs 5 less gas than i++

Using the former will help optimize gas costs, especially in for loops.

```
85        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```

## ```<=``` costs less gas than ```<```

This is because ```<``` uses the extra ```ISZERO``` opcode, which costs 3 gas. Consider replacing ```<``` occurrences with ```<=```

```
87            transferIndex < Math.max(sourcesLength, targetsLength);

90            address source = transferIndex < sourcesLength

93            address target = transferIndex < targetsLength

101            } else if (transferIndex < sourcesLength) {

104            } else if (transferIndex < targetsLength) {
```

## Using hardcoded address instead address(this) saves gas

```
209                address(this),
```

## Use assembly to hash instead of solidity

This will save approximately 80 gas per hash. 

```
206        bytes32 hash = keccak256(

211        keccak256(bytecode)
```
