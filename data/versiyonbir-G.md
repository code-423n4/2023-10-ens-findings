# STORAGE VARIABLE CACHING IN MEMORY
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L28-L28

The contract ERC20MultiDelegate is using the state variable token multiple times in the function _reimburse.
The contract ERC20MultiDelegate is using the state variable token multiple times in the function transferBetweenDelegators.
The contract ERC20MultiDelegate is using the state variable token multiple times in the function deployProxyDelegatorIfNeeded.

SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).

Storage variables read multiple times inside a function should instead be cached in the memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.


# CHEAPER CONDITIONAL OPERATORS
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L75-L75
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L110-L110
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L113-L113

During compilation, x != 0 is cheaper than x > 0 for unsigned integers in solidity inside conditional statements.

Consider using x != 0 in place of x > 0 in uint wherever possible.

# GAS OPTIMIZATION IN INCREMENTS
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L88-L88

++i costs less gas compared to i++ or i += 1 for unsigned integers. In i++, the compiler has to create a temporary variable to store the initial value. This is not the case with ++i in which the value is directly incremented and returned, thus, making it a cheaper alternative.

Consider changing the post-increments (i++) to pre-increments (++i) as long as the value is not used in any calculations or inside returns. Make sure that the logic of the code is not changed.

# LONG REQUIRE/REVERT STRINGS
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74-L77
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79-L82

The require() and revert() functions take an input string to show errors if the validation fails.
This strings inside these functions that are longer than 32 bytes require at least one additional MSTORE, along with additional overhead for computing memory offset, and other parameters.

It is recommended to short the strings passed inside require() and revert() to fit under 32 bytes. This will decrease the gas usage at the time of deployment and at runtime when the validation condition is met.

# CHEAPER INEQUALITIES IN IF()
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L110-L110
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L113-L113

The contract was found to be doing comparisons using inequalities inside the if statement.
When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <).

It is recommended to go through the code logic, and, if possible, modify the strict inequalities with the non-strict ones to save ~3 gas as long as the logic of the code is not affected.

# UNNECESSARY CHECKED ARITHMETIC IN LOOP
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L88-L88

Increments inside a loop could never overflow due to the fact that the transaction will run out of gas before the variable reaches its limits. Therefore, it makes no sense to have checked arithmetic in such a place.

It is recommended to have the increment value inside the unchecked block to save some gas.

# DEFINE CONSTRUCTOR AS PAYABLE

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L16-L19
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L44-L49

Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.

It is suggested to mark the constructors as payable to save some gas. Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.

# ABI ENCODE IS LESS EFFICIENT THAN ABI ENCODEPACKED
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L204-L204

The contract is using abi.encode() in the function retrieveProxyContractAddress. In abi.encode(), all elementary types are padded to 32 bytes and dynamic arrays include their length, whereas abi.encodePacked() will only use the minimal required memory to encode the data.

Unless explicitly needed , it is recommended to use abi.encodePacked() instead of abi.encode().
