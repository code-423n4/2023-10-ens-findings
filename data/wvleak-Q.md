## [L-01]: Block Gas Limit Vulnerability

### Severity

**Impact:** Low

**Likelihood:** Medium

### Description

The creation of contracts is an operation that consumes a significant amount of gas. In the `delegateMulti()` function, there is a for loop that uses unbounded arrays. If this function is used to create multiple contracts during the deployment of the proxy delegator, it could easily exceed the block gas limit, preventing transactions from completing.

**Code Reference:** [ERC20MultiDelegate.sol#L65](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65)

```solidity
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

       ...

        // This loop can be unbounded
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            ... // Unbounded loop
        }

        ...
    }
```

### Recommendations

When using a loop to iterate over an array of unknown size, it is important to plan for the possibility that it might span multiple blocks and, consequently, require multiple transactions. In this case, it is advisable to limit the size of the arrays to a fixed maximum value.

---

## [L-02]: Delegation Redundancy and Log Bloating

### Severity

**Impact:** Low

**Likelihood:** Medium

### Description

In the `_processDelegation()` function, there is no check to verify if the source and target addresses are the same when transferring delegation. This oversight allows redundant events to be emitted repeatedly, especially when the `delegateMulti()` function iterates through unbounded arrays.

**Code Reference:** [ERC20MultiDelegate.sol#L124](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124)

### Exploit Scenario

An attacker can exploit this by calling `delegateMulti()` with a large array size (e.g., hundreds) of source delegatees pointing to the same target address. As a result, an event is emitted for each iteration, causing the event logs to bloat with unnecessary information. This negatively impacts the contract's performance and off-chain monitoring systems.


This forge test showcase the feasibility of the scenario:
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../contracts/ERC20MultiDelegate.sol";
import "../contracts/ENSToken.sol";

contract DelegateTest is Test {
    ERC20MultiDelegate public multiDelegate;
    ENSToken public token;
    uint256[] public sources;
    uint256[] public delegates;
    uint256[] public amounts;

    function setUp() public {
        token = new ENSToken(20000 ether, 0, 0);
        multiDelegate = new ERC20MultiDelegate(token, "");
        token.transfer(address(1), 10000 ether);
        for (uint i; i < 1000; i++) {
            delegates.push(i + 2);
            sources.push(i + 2);
            amounts.push(1 ether);
        }
    }

    function testLogBloat() public {

        vm.startPrank(address(1));
        token.approve(address(multiDelegate), type(uint256).max);

        //User delegate from same sources and delgates
        multiDelegate.delegateMulti(sources, delegates, amounts);

        assertEq(multiDelegate.balanceOf(address(1), delegates[0]), 1 ether);
    }
}

```

### Recommendations

To address this issue, it is recommended to add a `require` statement in the `_processDelegation()` function to check whether the source and target addresses are different, preventing redundancy in event emissions.

```solidity
function _processDelegation(
    address source,
    address target,
    uint256 amount
) internal {
    uint256 balance = getBalanceForDelegate(source);
    require(source != target, "Target is the same as Source"); // Prevent redundancy
    assert(amount <= balance);

    deployProxyDelegatorIfNeeded(target);
    transferBetweenDelegators(source, target, amount);

    emit DelegationProcessed(source, target, amount);
}
```

---

## [L-03]: Missing Events for Reimbursement and First Delegation

### Severity

**Impact:** Low

### Description

Currently, events are only emitted when delegation is transferred using the `DelegationProcessed` event in the `_processDelegation()` function. However, no events are emitted for delegation reimbursement or delegation to a new user in the `_reimburse()` and `createProxyDelegatorAndTransfer()` functions.

**Code References:** [_reimburse](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144) and [createProxyDelegatorAndTransfer](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155)

### Recommendations

For improved off-chain monitoring, consider emitting specific events for delegation reimbursement and the creation of proxy delegators in the `_reimburse()` and `createProxyDelegatorAndTransfer()` functions, respectively.

---

## [NC-01]: Unused Library Methods

### Severity

**Impact:** Non-Critical

### Description

The `Address` library methods are imported but never used in the code.

### Recommendations

To enhance code clarity and save deployment gas costs, consider removing these unused `Address` library import.

---

## [N-02]: Revert Earlier with Appropriate Checks

### Severity

**Impact:** Non-critical

### Description

Several checks in the code are made after the completion of a loop in the `delegateMulti()` function. For example, the verification of the source balance for a user occurs after the entire loop using `_burnBatch()`. Checking that a user has a sufficient amount of ENSToken before transfer in the `createProxyDelegatorAndTransfer()` function could also be performed earlier. Reverting earlier in these cases can improve code maintainability and gas efficiency.

**Code References:** [ERC20MultiDelegate.sol#L111](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L111), [ERC20MultiDelegate.sol#L144](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144), and [ERC20MultiDelegate.sol#L160](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160)

### Recommendations

Consider adding the checks for source balance and other conditions earlier in the code to increase code maintainability and gas efficiency.

- Add check in `_reimburse()` for source balance:
```solidity
function _reimburse(address source, uint256 amount) internal {
        require(getBalanceForDelegate(source) >= amount, "Insufficient source balance") //@audit
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount); //@audit-issue - unchecked return value
    }
```

- Delegator amount can be checked too in `createProxyDelegatorAndTransfer()` (less significant).
