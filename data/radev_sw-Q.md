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


---

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

### Event for `delegateMulti()` / `_delegateMulti()`

For a function that is dealing with multiple delegations, it would be handy to have events that provide a clear indication of each delegation that took place:

```solidity
event Delegated(address indexed source, address indexed target, uint256 amount);
```

This event will be emitted within the `_processDelegation` function each time a delegation takes place. If `_processDelegation` is a function in the same contract, you would insert `emit Delegated(source, target, amount);` within that function.

---

### Event for `_reimburse()`

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

### Event for `createProxyDelegatorAndTransfer()`

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