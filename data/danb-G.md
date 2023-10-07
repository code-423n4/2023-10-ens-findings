# Optimizations
## Optimizing the proxy addresses
The `retrieveProxyContractAddress` function is called multiple times for each of the "main loop"'s iterations.
I propose 2 possible different ways to fixe for this issue:
* Save a mapping between each delegate to the "Proxy Contact address" and change the `retrieveProxyContractAddress` function to return the value from the mapping if it exists, and calculate it and save it to the mapping for future lookup otherwise 
* Calculate the "Proxy Contact address" once ahead of time in the beginning of the `_delegateMulti` function and pass it to the functions that require it
## Optimizing the loop in the main function
the loop in the main function has multiple optimizations which could be implemented:
* The `Math.max(sourcesLength, targetsLength)` condition in the main loop should be calculated before it like so:
```
uint256 stopCondition = Math.max(sourcesLength, targetsLength)
for (uint transferIndex =  0; transferIndex < stopCondition; transferIndex++)
```
* Additionally, the main loop should be split into two separate loops in order to minimize the calculation of the conditions which compare the indexes to the sizes of the input arrays in the for loop like so:
```
uint256 stopConditionFirstLoop = Math.min(sourcesLength, targetsLength);
uint256 stopConditionSecondLoop = Math.max(sourcesLength, targetsLength);
uint transferIndex =  0;
for (; transferIndex < stopConditionFirstLoop ; transferIndex++) {
	address source = transferIndex < sourcesLength
	?  address(uint160(sources[transferIndex]))
	:  address(0);
	
	address target = transferIndex < targetsLength
	?  address(uint160(targets[transferIndex]))
	:  address(0);
	uint256 amount = amounts[transferIndex];

	_processDelegation(source, target, amount);
}

if (transferIndex < sourcesLength) {
	for (; transferIndex < stopConditionSecondLoop; transferIndex++) {
		address source = transferIndex < sourcesLength
		?  address(uint160(sources[transferIndex]))
		:  address(0);
		
		uint256 amount = amounts[transferIndex];

		_reimburse(source, amount);
	}
} else  if (transferIndex < targetsLength) {
	for (; transferIndex < stopConditionSecondLoop; transferIndex++) {
		address target = transferIndex < targetsLength
		?  address(uint160(targets[transferIndex]))
		:  address(0);
		
		uint256 amount = amounts[transferIndex];
			
		createProxyDelegatorAndTransfer(target, amount);
	}
}
```