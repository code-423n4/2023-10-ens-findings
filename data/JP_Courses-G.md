0. ERC20MultiDelegate::_delegateMulti() - Using `!= 0` instead of `> 0` in Solidity/EVM is more gas-efficient due to lower gas costs and enhances code readability.

https://github.com/code-423n4/2023-10-ens/blob/ed47c841a19abd26681110a26ef03c446da2b6dd/contracts/ERC20MultiDelegate.sol#L74-L77

DESCRIPTION/PoC:

In Solidity and the Ethereum Virtual Machine (EVM), it's more gas-efficient to use `!= 0` instead of `> 0` in certain cases due to the following reasons:

- Gas Cost: EVM's gas costs for arithmetic operations increase with the magnitude of numbers involved and checking for equality (`!=`) is less gas-expensive compared to checking for inequality (`>`, `<`, `>=`, or `<=`).

- Readability: Using `!= 0` is more straightforward and intuitive for other developers, conveying the intention of checking for non-zero values.

In summary, using `!= 0` optimizes gas usage, making smart contracts more cost-effective and readable, which is crucial for complex contracts and high-frequency operations.

Recommendation:
```solidity
        require(
        --  sourcesLength > 0 || targetsLength > 0,
        ++  sourcesLength != 0 || targetsLength != 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
```

1. ERC20MultiDelegate::_delegateMulti() - Reusing a cached value for `Math.max(sourcesLength, targetsLength)` in the require() statement and `for` loop condition minimizes gas costs associated with repeated calculations, optimizing gas efficiency in the smart contract.

https://github.com/code-423n4/2023-10-ens/blob/ed47c841a19abd26681110a26ef03c446da2b6dd/contracts/ERC20MultiDelegate.sol#L79-L89

RECOMMENDATION:
(with some additional gas optimizations included here since it's all related to same code blocks)

I suggest to use a cached value here by adding a variable to hold the result of `Math.max(sourcesLength, targetsLength)` in memory and then reuse this cached value in the loop condition too. 

Here `maxSourcesOrTargets` is introduced to store the maximum of `sourcesLength` and `targetsLength`. 
This value is calculated only once, and then it's used in both the `require` statement and the loop condition. 

By reusing this cached value, you minimize the gas costs associated with repeated calculations of `Math.max(sourcesLength, targetsLength)` during the require check & especially the `for` loop's execution, making the code more efficient and cost-effective.

L79-L89:
```solidity
    ++  // Calculate the maximum of sourcesLength and targetsLength and cache it.
    ++  uint256 maxSourcesOrTargets = Math.max(sourcesLength, targetsLength);
    
        require(
        --  Math.max(sourcesLength, targetsLength) == amountsLength,
        ++  maxSourcesOrTargets == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
        --  uint transferIndex = 0;
        ++  uint transferIndex;
        --  transferIndex < Math.max(sourcesLength, targetsLength);
        ++  transferIndex < maxSourcesOrTargets; 
        --  transferIndex++
        ) {
            // loop logic
    				// ...
    		++  unchecked {
    		++      transferIndex++;
    		++  }
				}
```
Cleaning up, it should look like this after the above changes:
```solidity
        // Calculate the maximum of sourcesLength and targetsLength and cache it.
        uint256 maxSourcesOrTargets = Math.max(sourcesLength, targetsLength);
    
        require(
            maxSourcesOrTargets == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex;
            transferIndex < maxSourcesOrTargets; 
        ) {
            // loop logic
    				// ...
    		    unchecked {
    		        transferIndex++;
    		    }
				}
```

2. ERC20MultiDelegate::_delegateMulti() - Reusing a cached value for `Math.min(sourcesLength, targetsLength)` in the `if` statement inside the `for` loop minimizes gas costs associated with repeated calculations, optimizing gas efficiency in the smart contract.

https://github.com/code-423n4/2023-10-ens/blob/ed47c841a19abd26681110a26ef03c446da2b6dd/contracts/ERC20MultiDelegate.sol#L98

Somewhat similar to the previous gas optimization finding, here we should cache the value for `Math.min(sourcesLength, targetsLength)` so that we can use the cached value instead in the `if` statement inside the `for` loop, which will eliminate the need for EVM to waste gas on recalculating the same `Math.min(sourcesLength, targetsLength)` over and over for potentially many `for` loop iterations.

RECOMMENDATION:

Suggested gas optimizations(excluding optimizations from previous finding):
L84-L98:
```solidity
    ++  // Calculate the minimum of sourcesLength and targetsLength and cache it.
    ++  uint256 minSourcesOrTargets = Math.min(sourcesLength, targetsLength);
    
        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++ 
        ) {
            // for loop logic
            //...

        --  if (transferIndex < Math.min(sourcesLength, targetsLength)) {
        ++  if (transferIndex < minSourcesOrTargets) {
```
And if we add this and the previous finding's optimizations together:
```solidity
        // Calculate the maximum of sourcesLength and targetsLength and cache it.
        uint256 maxSourcesOrTargets = Math.max(sourcesLength, targetsLength);
    
        require(
            maxSourcesOrTargets == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

    ++  // Calculate the minimum of sourcesLength and targetsLength and cache it.
    ++  uint256 minSourcesOrTargets = Math.min(sourcesLength, targetsLength);
    
        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex;
            transferIndex < maxSourcesOrTargets; 
        ) {
            // for loop logic
    				// ...
    				
        ++  if (transferIndex < minSourcesOrTargets) {
    				    // if block logic
    				    //...
    				}
    				
    		    unchecked {
    		        transferIndex++;
    		    }
				}
```

3. (Please see `1.` from my QA report, as that finding is both a gas optimization and QA/LOW finding)
