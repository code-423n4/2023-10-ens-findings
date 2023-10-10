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

| ID                | Title                                                   | Instances | Severity |
| ----------------- | ------------------------------------------------------- | --------- | -------- |
| [GAS-01](#GAS-01) | Use of `Math.max` and `Math.min`                        | 1         | _Low_    |
| [GAS-02](#GAS-02) | Single Read of `Math.max(sourcesLength, targetsLength)` | 1         | _Low_    |
| [GAS-03](#GAS-03) | Single Read of `Math.min(sourcesLength, targetsLength)` | 1         | _Low_    |
| [GAS-04](#GAS-04) | Use of Indexed Events                                   | 1         | _Low_    |

---

---

## <a name="GAS-01"></a>[GAS-01] Use of `Math.max` and `Math.min`

### Overview:

The contract currently uses the OpenZeppelin `Math` library's `max` and `min` functions to determine the maximum and minimum of `sourcesLength` and `targetsLength`. Although OpenZeppelin libraries are known for their robustness, certain simple operations like determining the maximum and minimum can be performed directly in the contract without the need for external calls. This can potentially lead to some gas savings.

### Code

```solidity
    function max(uint256 a, uint256 b) internal pure returns (uint256) {
        return a >= b ? a : b;
    }

    function min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a <= b ? a : b;
    }
```

### Recommendation:

Implement custom `max` and `min` utility functions within the contract to replace external calls to the `Math` library for these specific operations.

---

## <a name="GAS-02"></a>[GAS-02] Single Read of `Math.max(sourcesLength, targetsLength)`

### Overview:

In the `_delegateMulti()` function, the value of `Math.max(sourcesLength, targetsLength)` is computed multiple times within a loop. Each computation involves gas costs, and therefore computing this value only once and saving it in memory could lead to gas savings.

### Code

```solidity
    uint256 maxLen = max(sourcesLength, targetsLength);

    for (uint transferIndex = 0; transferIndex < maxLen; transferIndex++) {
        // ...
    }
```

### Recommendation:

Store the value of `Math.max(sourcesLength, targetsLength)` in a local variable at the beginning of the function and reference this variable wherever needed in the function.

---

## <a name="GAS-03"></a>[GAS-03] Single Read of `Math.min(sourcesLength, targetsLength)`

### Overview:

Similar to the above issue, the value of `Math.min(sourcesLength, targetsLength)` is also computed multiple times within a loop in the `_delegateMulti()` function. This repetitive computation can be avoided to save gas.

### Code

```solidity
    uint256 minLen = min(sourcesLength, targetsLength);

    if (transferIndex < minLen) {
        // ...
    }
```

### Recommendation:

Store the value of `Math.min(sourcesLength, targetsLength)` in a local variable at the beginning of the function and reference this variable in the necessary places.

---

## <a name="GAS-04"></a>[GAS-04] Use of Indexed Events

### Overview:

By indexing event parameters, they are not stored in the event data but in a separate data structure called the topics array, which can save a considerable amount of gas.
Using indexed parameters in events can help reduce the gas costs when filtering for specific event logs. In the given contract, certain important parameters in the events such as `delegate` and `from` are already indexed, which is good. However, ensuring that all necessary fields are indexed can further optimize gas usage.

### Code

```solidity
    event SomeEvent(address indexed user, uint256 indexed value, ...);
```

### Recommendation:

Review all events and ensure that all parameters that users might want to filter by are indexed.

---
