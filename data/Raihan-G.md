# Gas

# Summary
|      |  issue   | insteance |
|------|----------|-----------|
|[G-01]|Can Make The Variable Outside The Loop To Save Gas |3|
|[G-02]|Make 3 event parameters indexed when possible|2|
|[G-03]|Use constants instead of type(uintx).max|3|
|[G-04]|Use assembly to perform efficient back-to-back calls|1|
|[G-05]|Use do while loops instead of for loops|1|
|[G-06]| Use `require` instead of `assert`|1|


## [G-01] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File:  contracts/ERC20MultiDelegate.sol
90   address source = transferIndex < sourcesLength

93   address target = transferIndex < targetsLength

96   uint256 amount = amounts[transferIndex];
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90

### `Fix code `

```diff
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
+    // Declare variables outside the loop to save gas
+       address source;
+       address target;
+       uint256 amount;

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
-            address source = transferIndex < sourcesLength
+            source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
-           address target = transferIndex < targetsLength
+           target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
-           uint256 amount = amounts[transferIndex];
+           amount = amounts[transferIndex];
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
## [G-02] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.
```solidity
File:  contracts/ERC20MultiDelegate.sol
32  event ProxyDeployed(address indexed delegate, address proxyAddress);
    event DelegationProcessed(
        address indexed from,
        address indexed to,
        uint256 amount
    );
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L32-L37

### `Fix code `
```diff
-    event ProxyDeployed(address indexed delegate, address proxyAddress);
+    event ProxyDeployed(address indexed delegate, address indexed proxyAddress);
    event DelegationProcessed(
        address indexed from,
        address indexed to,
-       uint256 amount
    );
    event DelegationProcessed(
        address indexed from,
        address indexed to,
+       uint256 indexed amount
    );
```
## [G-03] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File:  contracts/ERC20MultiDelegate.sol
17    _token.approve(msg.sender, type(uint256).max);
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17


### `Fix code `
```diff
contract ERC20ProxyDelegator {
+   uint256 constant MAX_UINT256 = 2**256 - 1;    
    constructor(ERC20Votes _token, address _delegate) {
-       _token.approve(msg.sender, type(uint256).max);
+       _token.approve(msg.sender, MAX_UINT256);
        _token.delegate(_delegate);
    }
}
```
## [G-04] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

```solidity
File:  contracts/ERC20MultiDelegate.sol
17      _token.approve(msg.sender, type(uint256).max);
18      _token.delegate(_delegate);
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17-L18


### `Fix code `
```diff
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
+        assembly {
+           // Prepare the data for the _token.approve call
+           let approveData := mload(0x40) // Get a free memory pointer
+           mstore(approveData, 0x095ea7b3000000000000000000000000) // Function selector for approve(address,uint256)
+           mstore(add(approveData, 0x04), caller()) // Set the address to approve as the contract caller
+           mstore(add(approveData, 0x24), not(0)) // Set the value to approve as maximum uint256

+           // Perform the _token.approve call
+           let success1 := call(gas(), _token, 0, approveData, 0x44, 0, 0)

+           // Prepare the data for the _token.delegate call
+           let delegateData := mload(0x40) // Get a free memory pointer
+           mstore(delegateData, 0xf8559f2d000000000000000000000000) // Function selector for delegate(address)
+           mstore(add(delegateData, 0x04), _delegate) // Set the delegate address

+           // Perform the _token.delegate call
+           let success2 := call(gas(), _token, 0, delegateData, 0x24, 0, 0)

+           // Set the zero slot back to 0 if necessary
+           if iszero(success1) {
+               mstore(0, 0)
+           }
+       }
+    }
}
```
## [G-05] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

```solidity
File:  contracts/ERC20MultiDelegate.sol
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85-L89


### `Fix code `
```diff
+   uint256 transferIndex;

+   do {
+       address source = transferIndex < sourcesLength
+           ? address(uint160(sources[transferIndex]))
+           : address(0);
+       address target = transferIndex < targetsLength
+           ? address(uint160(targets[transferIndex]))
+           : address(0);
+       uint256 amount = amounts[transferIndex];

+       if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+           // Process the delegation transfer between the current source and target delegate pair.
+           _processDelegation(source, target, amount);
+       } else if (transferIndex < sourcesLength) {
+           // Handle any remaining source amounts after the transfer process.
+           _reimburse(source, amount);
+       } else if (transferIndex < targetsLength) {
+           // Handle any remaining target amounts after the transfer process.
+           createProxyDelegatorAndTransfer(target, amount);
+       }

+       transferIndex++;
+   } while (transferIndex < Math.max(sourcesLength, targetsLength));
```
## [G-06] Use `require` instead of `assert`
The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.
[Reffrence](https://solodit.xyz/issues/g-01-use-code4rena-zksync-zksync-v2-contest-git)
```solidity
File:  contracts/ERC20MultiDelegate.sol
131  assert(amount <= balance);
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131