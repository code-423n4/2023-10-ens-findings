# Gas Optimizations Report - ENS Audit Contest | 5 Oct 2023 - 11 Oct 2023

---

# Executive summary

### Overview

|              |                                           |
| :----------- | :---------------------------------------- |
| Project Name | ENS                                       |
| Repository   | https://github.com/code-423n4/2023-10-ens |
| Website      | [Link](https://ens.domains/)              |
| Twitter      | [Link](https://twitter.com/ensdomains)    |
| Methods      | Manual review                             |

### Scope

| Contract (1)                                                                                                             | SLOC | Purpose                                                                | Libraries used                                           |
| :----------------------------------------------------------------------------------------------------------------------- | :--- | :--------------------------------------------------------------------- | :------------------------------------------------------- |
| [contracts/ERC20MultiDelegate.sol](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) | 216  | ERC20Votes compatible multi-delegation contract to manage user votings | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

---

---

# Findings Summary

| ID                | Title                                                   |
| ----------------- | ------------------------------------------------------- |
| [GAS-01](#GAS-01) | Use of `Math.max` and `Math.min`                        |
| [GAS-02](#GAS-02) | Single Read of `Math.max(sourcesLength, targetsLength)` |
| [GAS-04](#GAS-04) | Use of Indexed Events                                   |
| [GAS-05](#GAS-05) | Use assembly for loops to save gas                      |

---

---

## <a name="GAS-01"></a>[GAS-01] Use of `Math.max` and `Math.min`

### Overview:

The contract currently uses the OpenZeppelin `Math` library's `max` and `min` functions to determine the maximum and minimum of `sourcesLength` and `targetsLength`. Although OpenZeppelin libraries are known for their robustness, certain simple operations like determining the maximum and minimum can be performed directly in the contract without the need for external calls. This can potentially lead to some gas savings.

### GitHub Links:

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98

### Example how to rewrite the code:

Implement custom `max` and `min` utility functions within the contract to replace external calls to the `Math` library for these specific operations.

```solidity
    function max(uint256 a, uint256 b) internal pure returns (uint256) {
        return a >= b ? a : b;
    }

    function min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a <= b ? a : b;
    }
```

---

## <a name="GAS-02"></a>[GAS-02] Single Read of `Math.max(sourcesLength, targetsLength)`

### Overview:

In the `_delegateMulti()` function, the value of `Math.max(sourcesLength, targetsLength)` is computed multiple times within a loop. Each computation involves gas costs, and therefore computing this value only once and saving it in memory could lead to gas savings.

### GitHub links:

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

### Example how to rewrite the code:

```jsx
+   uint256 maxLength = Math.max(sourcesLength, targetsLength);

+   for (uint transferIndex; transferIndex < maxLength; transferIndex++) {
        // ...
    }
```

### Recommendation:

Store the value of `Math.max(sourcesLength, targetsLength)` in a local variable at the beginning of the function and reference this variable wherever needed in the function.

---

## <a name="GAS-04"></a>[GAS-04] Use of Indexed Events

### Overview:

By indexing event parameters, they are not stored in the event data but in a separate data structure called the topics array, which can save a considerable amount of gas.
Using indexed parameters in events can help reduce the gas costs when filtering for specific event logs. In the given contract, certain important parameters in the events such as `delegate` and `from` are already indexed, which is good. However, ensuring that all necessary fields are indexed can further optimize gas usage.

### GitHub links:

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L32-L37

### Example how the code to be rewrite:

```jsx
+   event ProxyDeployed(address indexed delegate, address indexed proxyAddress);
    event DelegationProcessed(
        address indexed from,
        address indexed to,
+       uint256 indexed amount
    );
```

### Recommendation:

Review all events and ensure that all parameters that users might want to filter by are indexed.

---

## <a name="GAS-05"></a>[GAS-05] Use assembly for loops to save gas

**Saves `2450 GAS` for every iteration from `7 instances`.
Assembly is more gas efficient for loops. Saves minimum `350 GAS` per iteration as per remix gas checks.**

```jsx
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

[Links to code](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116)
