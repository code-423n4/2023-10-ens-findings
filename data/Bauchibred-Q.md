# QA Report

## **Table of Contents**

|       | Issue                                                                                           |
| ----- | ----------------------------------------------------------------------------------------------- |
| QA-01 | `setUri()` Should Emit An Event                                                                 |
| QA-02 | Clearly Add Comment to Instances of Magic Numbers                                               |
| QA-03 | Consider Making `ERC20ProxyDelegator` Upgradeable                                               |
| QA-04 | Descriptive Naming Conventions Should be Adopted                                                |
| QA-05 | `_delegateMulti()` Should be Made More Effective                                                |
| QA-06 | Functions Should Always be Documented                                                           |
| QA-07 | Fix Missing Input Validations                                                                   |
| QA-08 | Static Salt Value for Proxy Deployment                                                          |
| QA-09 | `_processDelegation()`, `_reimburse()` & `createProxyDelegatorAndTransfer()` Should Emit Events |
| QA-10 | Using Multiple Loops                                                                            |
| QA-11 | Consider Adding Circuit Breakers                                                                |
| QA-12 | Assembly Codes Should Always be Accompanied With Comments                                       |

## QA-01 `setUri()` Should Emit An Event

### Impact

Difficult to track changes, and users wouldn't know what `uri` this points to without checking it out which might be dangerous cause if a malicious link is set, users must click this to find out.

### Proof of Concept

Take a look at `setUri()`

```solidity
    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }

    //which internally calls thia:

    function _setURI(string memory newuri) internal virtual {
        _uri = newuri;
    }

```

As seen when the URI is changed using `setUri`, no event is emitted.

### Recommended Mitigation Steps

Emit an event indicating the change in URI in the `setUri` function.

## QA-02 Clearly Add Comment to Instances of Magic Numbers

### Impact

Low/info, difficulty in understanding the significance of certain values.

### Proof of Concept

Take a look at `retrieveProxyContractAddress()`

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

As seen `0xff` is used without context.

### Recommended Mitigation Steps

Provide clear comments or use named constants for magic numbers.

## QA-03 Consider Making `ERC20ProxyDelegator` Upgradeable

### Impact

Low/info, most proxies as the name suggest are upgradeable, so as to add in new logics or address issues after deployment but that's currently not posssible.

### Proof of Concept

Once the `ERC20ProxyDelegator` is deployed, there's no way to upgrade or modify it.

### Recommended Mitigation Steps

Consider using a proxy pattern that allows for upgrades.

## QA-04 Descriptive Naming Conventions Should be Adopted

### Impact

Low, info Can lead to developer confusion.

### Proof of Concept

As an example take a look at `_reimburse()`

```solidity
    function _reimburse(address source, uint256 amount) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

As seen, function names `_reimburse`like might be misinterpreted. A more explicit name like `_reimburseToSender` could be clearer.

### Recommended Mitigation Steps

Rename functions to more descriptive variants.

## QA-05 `_delegateMulti()` Should be Made More Effective

---

### Impact

Low, idea for refactoring code for better structure.

The loop condition inside the `_delegateMulti` function is potentially inefficient due to a redundant check. This can result in suboptimal gas usage during execution, which can lead to increased transaction costs.

---

### Proof of Concept

In the loop iteration:

```solidity
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
)
```

The call to `Math.max(sourcesLength, targetsLength)` is unnecessary because, due to a preceding validation check:

```solidity
require(
    Math.max(sourcesLength, targetsLength) == amountsLength,
    "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
);
```

it is guaranteed that `Math.max(sourcesLength, targetsLength)` is always equal to `amountsLength`.

So the use of `Math.max(sourcesLength, targetsLength)` in a loop condition can be replaced by `amountsLength` due to the check `Math.max(sourcesLength, targetsLength) == amountsLength`.

### Recommended Mitigation Steps

Replace the loop condition:

```solidity
transferIndex < Math.max(sourcesLength, targetsLength);
```

with:

```solidity
transferIndex < amountsLength;
```

This change will optimize the loop condition, potentially saving gas during execution.

## QA-06 Functions Should Always be Documented

### Impact

Low.. info on how to better structure of code to help stop potential misunderstanding of functions.

### Proof of Concept

Unlike `_reimburse`, functions like `createProxyDelegatorAndTransfer()`, `setUri()`, `retrieveProxyContractAddress()` `getBalanceForDelegate()`, `deployProxyDelegatorIfNeeded()`, `transferBetweenDelegators()` don't have associated natspec comments.

### Recommended Mitigation Steps

Add detailed natspec comments for every function.

## QA-07 Fix Missing Input Validations

### Impact

Potential unexpected behavior.

### Proof of Concept

Inputs for the functions are not thoroughly validated. For instance, the `_delegateMulti` does not prevent an address from appearing in both `sources` and `targets`, where as the inherited contract's functions would revert the transaction's execution, users would not know what's wrong and lead them to have a bad UI/UX experience and frustrations

### Recommended Mitigation Steps

Introduce strict validation of inputs to functions, ensuring no overlaps or inconsistencies.

## QA-08 Static Salt Value for Proxy Deployment

### Impact

Low.. In general having a static salt causes a transaction to revert when it's been called in a tx before another call to it, which is why it could be a painless thing for an attacker to always front run a call to use a static salt and cause a bad UI/UX experience for users, this also limits flexibility and can cause issues.

### Proof of Concept

In `deployProxyDelegatorIfNeeded`:

```solidity
new ERC20ProxyDelegator{salt: 0}(token, delegate);
```

Looking at `retrieveProxyContractAddress()`

```
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

Both instances have the salt hardcoded and being statically set to `0`.

### Recommended Mitigation Steps

Allow dynamic salt values or use unique values derived from delegate addresses, i.e the idea should be to use an ever increasing value for the nonce

## QA-09 `_processDelegation()`, `_reimburse()` & `createProxyDelegatorAndTransfer()` Should Emit Events

### Impact

Low, current implementation makes direct tracking token movements harder.

### Proof of Concept

No event is emitted in all three functions: `_processDelegation()`, `_reimburse()` & `createProxyDelegatorAndTransfer()` after tokens are transferred.
Do note that whereas these functions are called from the `delegateMulti()`

```
    function delegateMulti() external {
        _delegateMulti(sources, targets, amounts);
    }

    function _delegateMulti() internal {
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

### Recommended Mitigation Steps

Add a suitable event and emit it after the token transfer in these functions

## QA-10 Using Multiple Loops

### Impact

Low

### Proof of Concept

Take a look at `_delegateMulti()`

```
    function _delegateMulti() internal {
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

As seen, multiple loops are used in `_delegateMulti` to handle different conditions. This can be gas inefficient.

### Recommended Mitigation Steps

Consider a more streamlined approach to reduce the number of loops and checks.

## QA-11 Consider Adding Circuit Breakers

### Impact

Info, currently if an issue arises, there's no way to pause contract interactions.

### Proof of Concept

Going through the `ERC20MultiDelegate.sol` contract, we can see that circuit breakers & pause functionalities are absent.

### Recommended Mitigation Steps

Consider adding a `Pausable` functionality from OpenZeppelin to allow for emergency stops, though if this would be implemented could lead to more centralization risks, but could come in handy.

## QA-12 Assembly Codes Should Always be Accompanied With Comments

### Impact

Difficulty in understanding and verifying assembly code.

### Proof of Concept

Take a look at `

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

Assembly is used in `deployProxyDelegatorIfNeeded` without comprehensive comments.

### Recommended Mitigation Steps

Always accompany inline assembly with comments explaining the logic and intent.
