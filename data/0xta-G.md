# Gas Optimization

# Summary

| Number | Gas Optimization                                                                      | Context |
| :----: | :------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Use constants instead of type(uintx).max                                              |    1    |
| [G-02] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |    2    |
| [G-03] | Don’t make variables public unless it is necessary to do so                           |    1    |
| [G-04] | Use hardcode address instead address(this)                                            |    1    |
| [G-05] | Using assembly to revert with an error message                                        |    2    |
| [G-06] | Use do while  loops instead of for loops                                              |    1    |
| [G-07] | Sort Solidity operations using short-circuit mode                                     |    1    |
| [G-08] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS   |    3    |
| [G-09] | Missing zero address checks in the constructor                                        |    1    |
| [G-10] | Consider using alternatives to OpenZeppelin                                           |         |

## [G-01] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

```solidity
File: contracts/ERC20MultiDelegate.sol

17        _token.approve(msg.sender, type(uint256).max);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol

## [G-02] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

It's recommended to use the bytes.concat() function instead of abi.encodePacked() for concatenating byte arrays. bytes.concat() is a more gas-efficient and safer way to concatenate byte arrays, and it's considered a best practice in newer Solidity versions.

```solidity
File: contracts/ERC20MultiDelegate.sol

202        bytes memory bytecode = abi.encodePacked(

207            abi.encodePacked(
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol

## [G-03] Don’t make variables public unless it is necessary to do so

A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.

```solidity
File: contracts/ERC20MultiDelegate.sol

28    ERC20Votes public token;
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L28

## [G-04] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

[Reference](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: contracts/ERC20MultiDelegate.sol

209                address(this),

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209

## [G-05] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

```solidity
File: contracts/ERC20MultiDelegate.sol

74       require(
75            sourcesLength > 0 || targetsLength > 0,
76            "Delegate: You should provide at least one source or one target delegate"
77        );



79        require(
80            Math.max(sourcesLength, targetsLength) == amountsLength,
81            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82        );
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74C1-L83C1

## [G-06] Use do while  loops instead of for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

```solidity
File: contracts/ERC20MultiDelegate.sol

85        for (
86            uint transferIndex = 0;
87            transferIndex < Math.max(sourcesLength, targetsLength);
88            transferIndex++
89        ) {
90            address source = transferIndex < sourcesLength
91                ? address(uint160(sources[transferIndex]))
92                : address(0);
93            address target = transferIndex < targetsLength
94                ? address(uint160(targets[transferIndex]))
95                : address(0);
96            uint256 amount = amounts[transferIndex];
97
98            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
99                // Process the delegation transfer between the current source and target delegate pair.
100                _processDelegation(source, target, amount);
101            } else if (transferIndex < sourcesLength) {
102                // Handle any remaining source amounts after the transfer process.
103                _reimburse(source, amount);
104            } else if (transferIndex < targetsLength) {
105                // Handle any remaining target amounts after the transfer process.
106                createProxyDelegatorAndTransfer(target, amount);
107            }
108        }

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85-L108

## [G-07] Sort Solidity operations using short-circuit mode

In Solidity, when you evaluate a boolean expression (e.g the || (logical or) or && (logical and) operators), in the case of || the second expression will only be evaluated if the first expression evaluates to false and in the case of && the second expression will only be evaluated if the first expression evaluates to true. This is called short-circuiting.

For example, the expression require(msg.sender == owner || msg.sender == manager) will pass if the first expression msg.sender == owner evaluates to true. The second expression msg.sender == manager will not be evaluated at all.

However, if the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will be evaluated to determine whether the overall expression is true or false. Here, by checking the condition that is most likely to pass firstly, we can avoid checking the second condition thereby saving gas in majority of successful calls.

This is similar for the expression require(msg.sender == owner && msg.sender == manager). If the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will not be evaluated because the overall expression cannot be true. For the overall statement to be true, both side of the expression must evaluate to true. Here, by checking the condition that is most likely to fail firstly, we can avoid checking the second condition thereby saving gas in majority of call reverts.

Short-circuiting is useful and it’s recommended to place the less expensive expression first, as the more costly one might be bypassed. If the second expression is more important than the first, it might be worth reversing their order so that the cheaper one gets evaluated first.

```solidity
File: contracts/ERC20MultiDelegate.sol

74        require(
75            sourcesLength > 0 || targetsLength > 0,
76            "Delegate: You should provide at least one source or one target delegate"
77        );
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74-L77

## [G-08] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity
File: contracts/ERC20MultiDelegate.sol

189        return proxyAddress;


195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));


214        return address(uint160(uint256(hash)));
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L189

## [G-09] Missing zero address checks in the constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity
File:

44    constructor(
45        ERC20Votes _token,
46        string memory _metadata_uri
47    ) ERC1155(_metadata_uri) {
48        token = _token;
49    }
```

## [G-10] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are [Solmate](https://github.com/transmissions11/solmate) and [Solady](https://github.com/Vectorized/solady).

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.