# QA Report - ENS Audit Contest | 5 Oct 2023 - 11 Oct 2023

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

| ID              | Title                                                                                                                                      | Instances | Severity                  |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------- | ------------------------- |
| [NC-01](#NC-01) | Potential Reversion Issue with Certain ERC20 Token Approvals                                                                               | 1         | _Non Critical_            |
| [NC-02](#NC-02) | Unchecked Return Values for `approve()`                                                                                                    | 1         | _Non Critical_            |
| [NC-03](#NC-03) | Need for Comments on State Variables                                                                                                       | 1         | _Non Critical_            |
| [NC-04](#NC-04) | Absence of Event Emissions                                                                                                                 | 3         | _Non Critical_            |
| [NC-05](#NC-05) | Missing address zero checks for `sources[]` and `targets[]` arrays in `delegateMulti()` function                                           | 1         | _Non Critical_            |
| [NC-06](#NC-06) | Unbounded `for` loop in `_delegateMulti()`. Set max value for the loop iterations (max values for `sourcesLength[]` and `targetsLength[]`) | 1         | _Non Critical_            |
| [S-01](#S-01)   | Optimization for `deployProxyDelegatorIfNeeded()` function logic                                                                           | -         | _Suggestion/Optimization_ |
| [S-02](#S-02)   | Optimization for transferring flow                                                                                                         | -         | _Suggestion/Optimization_ |

---

### <a name="NC-01"></a>[NC-01] Potential Reversion Issue with Certain ERC20 Token Approvals

Several well-known ERC20 tokens, such as `UNI` and `COMP`, may revert when the `approve` functions are called with values exceeding `uint96`. This behavior deviates from the standard ERC20 implementation and might lead to unexpected results.

For instance, both `UNI` and `COMP` tokens feature special-case logic in their `approve` functions. When the approval amount is `uint256(-1)`, the `allowance` is set to `type(uint96).max`. Systems that anticipate the approved value to match the value in the `allowances` mapping might face compatibility issues.

<i>This issue is present in the following location:</i>

```solidity
📁 File: contracts/ERC20MultiDelegate.sol

17:         _token.approve(msg.sender, type(uint256).max);
```

[Link to Code Line 17](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L17)

**Full Contract Context**:

```solidity
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}
```

---

### <a name="NC-02"></a>[NC-02] Unchecked Return Values for `approve()`

It is essential to note that not all `IERC20` implementations utilize the `revert()` mechanism to indicate failures in the `approve()` function. Some instead return a boolean to signal any issues. By not verifying this returned value, certain operations might proceed as if they were successful, despite no actual approvals taking place.

<i>The issue occurs in the following segment:</i>

```solidity
📁 File: contracts/ERC20MultiDelegate.sol

17:         _token.approve(msg.sender, type(uint256).max);
```

[Link to Code Line 17](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L17-L17)

---

### <a name="NC-03"></a>[NC-03] Need for Comments on State Variables

To enhance readability and assist future reviewers, it is advisable to provide detailed comments on crucial state variables. Such documentation clarifies their intended roles and usage.

<i>This recommendation is based on the following code:</i>

```solidity
📁 File: contracts/ERC20MultiDelegate.sol

28:     ERC20Votes public token;
```

[Link to Code Line 28](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L28-L28)

---

### <a name="NC-04"></a>[NC-04] Absence of Event Emissions

Event emissions play a vital role in tracking and auditing on-chain activities. The following functions currently lack event emissions, which may hinder transparency and monitoring:

- [`delegateMulti() / _delegateMulti()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116)
- [`reimburse()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144-L149)
- [`createProxyDelegatorAndTransfer()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155-L161)

Consider introducing appropriate events to facilitate a more transparent operation flow.

---

#### Event for `delegateMulti()` / `_delegateMulti()`

For a function that is dealing with multiple delegations, it would be handy to have events that provide a clear indication of each delegation that took place:

```solidity
event Delegated(address indexed source, address indexed target, uint256 amount);
```

This event will be emitted within the `_processDelegation` function each time a delegation takes place. If `_processDelegation` is a function in the same contract, you would insert `emit Delegated(source, target, amount);` within that function.

---

#### Event for `_reimburse()`

This function essentially sends the un-delegated or remaining tokens back to the user. An event for this would look something like:

```solidity
event Reimbursed(address indexed source, address indexed user, uint256 amount);
```

You would emit this event within the `_reimburse` function right after the `transferFrom`:

```solidity
token.transferFrom(proxyAddressFrom, msg.sender, amount);
emit Reimbursed(source, msg.sender, amount);
```

---

#### Event for `createProxyDelegatorAndTransfer()`

This function seems to be creating a proxy and then transferring a specified amount of tokens to this proxy. The event for this would look like:

```solidity
event ProxyCreatedAndFunded(address indexed target, address proxyAddress, uint256 amount);
```

You would emit this event within the `createProxyDelegatorAndTransfer` function right after the `transferFrom`:

```solidity
token.transferFrom(msg.sender, proxyAddress, amount);
emit ProxyCreatedAndFunded(target, proxyAddress, amount);
```

---

### <a name="NC-05"></a>[NC-05] Missing address zero checks for `sources[]` and `targets[]` arrays in `delegateMulti()` function

#### Problem:

Within the `delegateMulti()` and `_delegateMulti()` functions, the `sources` and `targets` arrays contain Ethereum addresses. These addresses are further used in delegation operations. However, there is currently no validation to prevent the inclusion of the Ethereum zero address (`address(0)`) within these arrays. Using the zero address can lead to unexpected behaviors or errors, especially in delegation scenarios.

#### Code Snippet:

```solidity
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
    ...
}
```

In the snippet above, there is logic to set `source` or `target` to `address(0)` if their respective lengths are exceeded. However, this does not prevent the user from manually providing the zero address in the `sources` or `targets` arrays.

#### Impact:

By not ensuring the absence of the Ethereum zero address in the `sources[]` and `targets[]` arrays, a number of potential risks arise:

1. **Irretrievable Token Loss:** If a user or developer mistakenly uses the zero address as a source or target, tokens may get sent to this address and become permanently inaccessible. This can lead to financial loss for users and a decrease in the total available supply of the token.
2. **Unexpected Behaviors:** Delegating to or from the zero address can lead to unpredictable results or system errors.

#### Suggested Resolution:

Introduce validation checks to ensure that none of the elements in the `sources[]` and `targets[]` arrays are equal to the Ethereum zero address. This can be done at the start of the `_delegateMulti()` function. The check can look something like:

```solidity
for (uint i = 0; i < sourcesLength; i++) {
    require(sources[i] != address(0), "Delegate: Source address cannot be the zero address");
}
for (uint j = 0; j < targetsLength; j++) {
    require(targets[j] != address(0), "Delegate: Target address cannot be the zero address");
}
```

By integrating this check, the contract can safeguard against accidental or malicious use of the Ethereum zero address within the delegation process.

---

### <a name="NC-06"></a>[NC-06] Unbounded `for` loop in `_delegateMulti()`: Set maximum limit for loop iterations (max values for `sourcesLength` and `targetsLength`)

#### Overview:

The function `_delegateMulti()` processes the delegation transfer process for multiple source and target delegates. Currently, there are no explicit bounds set for the lengths of the `sources` and `targets` arrays. This lack of upper limit poses potential risks in terms of gas costs and unwanted system behaviors.

#### Impact:

1. **Gas Costs:** If an excessively large number of delegates is supplied to the function, the unbounded loop can consume a lot of gas. In extreme cases, this might cause the transaction to fail due to reaching the block gas limit.

2. **Denial-of-Service (DoS) Attack Vector:** Malicious actors can attempt to exhaust the contract's functions by calling `_delegateMulti()` with a large number of delegates, potentially causing regular users' transactions to fail or become more expensive.

3. **Operational Delays:** For a contract that is intended to process other transactions in a timely manner, long-loop iterations can cause operational delays.

#### Recommendation:

To mitigate these risks, consider adding an explicit maximum bound for the lengths of the `sources[]` and `targets[]` arrays. This will ensure that the loop does not iterate an excessive number of times:

```solidity
uint256 constant MAX_DELEGATES = 100; // Example value; adjust as needed

require(sourcesLength <= MAX_DELEGATES, "Delegate: Too many source delegates provided");
require(targetsLength <= MAX_DELEGATES, "Delegate: Too many target delegates provided");
```

By introducing these checks, you can ensure that the `_delegateMulti()` function remains efficient and less susceptible to potential abuse.

---

---

### <a name="S-01"></a>[S-01] Optimization for `deployProxyDelegatorIfNeeded()` function logic

#### Optimization:

If the goal is only ever to retrieve the address (and not necessarily deploy every time), separating the retrieval logic and the deployment logic might be more efficient.

#### Code Snippet:

- **[deployProxyDelegatorIfNeeded()](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190) function:**

```solidity
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```

- **[retrieveProxyContractAddress()](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L198-L215) function:**

```solidity
    function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode,
            abi.encode(_token, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }
```

### <a name="S-02"></a>[S-02] Optimization for transferring flow

#### Overview:

During the delegation transfer process for multiple source and target delegates, it's essential to maintain an efficient process to save on gas costs and optimize the overall transaction process. It's been identified that the current transfer flow can be optimized by introducing two additional checks:

- Ensure that `source` and `target` delegates are different.
- Ensure that the amount to be transferred is greater than zero.

#### Benefits of Optimization:

1. **Gas Efficiency:** Avoiding unnecessary transfers can save on gas costs, as each unnecessary operation within the Ethereum Virtual Machine (EVM) incurs gas costs.

2. **Logical Integrity:** Transferring tokens between identical source and target addresses or transferring an amount of zero doesn't align with the logical intent of a transfer operation.

3. **Preventing Unexpected Behaviors:** Ensuring logical checks can prevent potential unwanted system behaviors, edge cases, or vulnerabilities.

#### Recommendation:

Incorporate the following checks before calling the functions that execute the actual transferring:

The modified section within the `_delegateMulti()` function can look like:

```solidity
// ... (rest of the code)

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
        if (source != target && amount > 0) {
            // Process the delegation transfer between the current source and target delegate pair.
            _processDelegation(source, target, amount);
        }
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        if (source != address(0) && amount > 0) {
            _reimburse(source, amount);
        }
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        if (target != address(0) && amount > 0) {
            createProxyDelegatorAndTransfer(target, amount);
        }
    }
}

// ... (rest of the code)

```

By implementing these checks, the contract becomes more optimized, reduces unnecessary operations, and ensures a logical flow of token transfers.

---
