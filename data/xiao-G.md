# summary

## Gas Optimizations

| NO     | Issue                             | Instance |
| ------ | --------------------------------- | -------- |
| [G-01] | Reduce unnecessary calculations   | 2        |
| [G-02] | Merge condition check             | 2        |
| [G-03] | Simplify loop iteration           | 1        |
| [G-04] | Avoid unnecessary function calls  | 1        |
| [G-05] | Simplify access to array elements | 1        |
| [G-06] | Avoid unnecessary assertions      | 1        |

## [G-01] Reduce unnecessary calculations

The original code uses intermediate variables sourcesLength, targetsLength and amountsLength to store the length of the array and then performs conditional checking. These intermediate variables are replaced by maxLen in the optimized code, thereby reducing unnecessary calculation and memory overhead.

```solidity
File: /contracts/ERC20MultiDelegate.sol

70      uint256 maxLen = Math.max(sources.length, targets.length);


132     if (amount > 0) {
            deployProxyDelegatorIfNeeded(target);
            transferBetweenDelegators(source, target, amount);
            emit DelegationProcessed(source, target, amount);
    }

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L70

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L132

## [G-02] Merge condition check

There are two separate condition checks in the original code to ensure that the arrays are not empty and the lengths match. The optimized code combines these two condition checks into one, thus reducing duplicate condition checks.

```solidity
File: /contracts/ERC20MultiDelegate.sol

74      require(
            maxLen > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
        require(
            amounts.length == maxLen,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74

## [G-03] Simplify loop iteration

Two loop variables transferIndex and i are used in the original code, one of which is a counter and the other is a variable used to index the array. The optimized code combines two variables into one i to reduce code complexity and improve readability

```solidity
File: /contracts/ERC20MultiDelegate.sol

85      for (uint256 i = 0; i < maxLen; i++) {
            address source = i < sources.length
                ? address(uint160(sources[i]))
                : address(0);
            address target = i < targets.length
                ? address(uint160(targets[i]))
                : address(0);
            uint256 amount = amounts[i];
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85

## [G-04] Avoid unnecessary function calls

The Math.max function was used in the original code to determine the upper limit for loop iterations. The optimized code directly uses the maxLen variable, thus avoiding unnecessary function calls and improving efficiency.

```solidity
File: /contracts/ERC20MultiDelegate.sol

85      for (uint256 i = 0; i < maxLen; i++) {
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85

## [G-05] Simplify access to array elements

The original code uses conditional statements to access elements of the source, target, and amount arrays, and determines whether to use the default value based on whether the index exceeds the length of the array. The optimized code uses index i directly to access array elements and only uses the default value when needed

```solidity
File: /contracts/ERC20MultiDelegate.sol

90      address source = i < sources.length
            ? address(uint160(sources[i]))
            : address(0);
        address target = i < targets.length
            ? address(uint160(targets[i]))
            : address(0);
        uint256 amount = amounts[i];
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90

## [G-06] Avoid unnecessary assertions

The assert statement checks the condition at runtime and causes the contract to abort if the condition is not met. If you can do proper condition checking in your code and return an error early on failure, instead of using assert, the gas consumption may be lower.

```solidity
File: /contracts/ERC20MultiDelegate.sol

131    require(amount <= balance, "Amount exceeds delegate balance");
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131
