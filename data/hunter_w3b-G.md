# Gas Optimization

Gas optimization techniques are critical in smart contract development to reduce transaction costs and make contracts more efficient. The ENS protocol, as observed in the provided contract, can benefit from various gas-saving strategies to enhance its performance. Below is a summary of gas optimization techniques followed by categorized findings within the contracts.

# Summary

| Finding                                                                                                      | Occurrences |
| :----------------------------------------------------------------------------------------------------------- | :---------: |
| Use clones or metaproxies when deploying very similar smart contracts that are not called frequently         |      1      |
| Do-While loops are cheaper than for loops                                                                    |      1      |
| Consider using alternatives to OpenZeppelin                                                                  |      4      |
| Using assembly to revert with an error message                                                               |      2      |
| Use assembly to perform efficient back-to-back calls                                                         |      2      |
| Missing zero address checks in the constructor                                                               |      1      |
| Use selfdestruct in the constructor if the contract is one-time use                                          |      2      |
| Use transfer hooks for tokens instead of initiating a transfer from the destination smart contract           |      3      |
| Use ERC2930 access list transactions when making cross-contract calls to pre-warm storage slots and contract |      3      |
| Use hardcode address instead address(this)                                                                   |      1      |
| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                        |      2      |
| Simple checks for zero can be done using assembly to save gas                                                |      1      |
| Do not reduce approval on transferFrom if current allowance is type(uint256).max                             |      3      |
| Avoid having token balances go to zero, always keep a small amount                                           |      1      |
| Use voting delegation as a gas saving measure                                                                |      1      |
| Don’t make variables public unless it is necessary to do so                                                  |      1      |
| Use branchless algorithms as a replacement for conditionals and loops                                        |      7      |
| Always use Named Returns                                                                                     |      3      |
| Use constants instead of type(uintx).max                                                                     |      1      |
| Short-circuit booleans                                                                                       |      1      |

## [G-01] Use clones or metaproxies when deploying very similar smart contracts that are not called frequently

When deploying multiple similar smart contracts, the gas costs can be high. To reduce these costs, you can use minimal clones or metaproxies which store the address of the implementation contract in their bytecode and interact with it as a proxy.

However, there is a trade-off between the runtime cost and deployment cost of clones. Clones are more expensive to interact with than normal contracts due to the delegatecall they use, so they should only be used when you don’t need to interact with them frequently. For example, the Gnosis Safe contract uses clones to reduce deployment costs.

EIP-1167: Minimal Proxy Standard

EIP-3448 Metaproxy Clone

`ERC20MultiDelegate::deployProxyDelegatorIfNeeded` Cheap Contract Deployment Through Clones

There's a way to save a significant amount of gas on deployment using Clones: [Resource](https://www.youtube.com/watch?v=3Mw-pMmJ7TA).

This is a solution that was adopted, as an example, by [Porter Finance](https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516). They realized that deploying using clones was 10x cheaper:

```solidity
File:

173    function deployProxyDelegatorIfNeeded(
174        address delegate
175    ) internal returns (address) {
176        address proxyAddress = retrieveProxyContractAddress(token, delegate);
177
178        // check if the proxy contract has already been deployed
179        uint bytecodeSize;
180        assembly {
181            bytecodeSize := extcodesize(proxyAddress)
182        }
183
184        // if the proxy contract has not been deployed, deploy it
185        if (bytecodeSize == 0) {
186            new ERC20ProxyDelegator{salt: 0}(token, delegate);//@audit gas: deployment can cost less through clones
187            emit ProxyDeployed(delegate, proxyAddress);
188        }
189        return proxyAddress;
190    }
```

There’s a way to save a significant amount of gas on deployment using Clones: [OpenZeppelin video](https://www.youtube.com/watch?v=3Mw-pMmJ7TA) This is a solution that was adopted, as an example, by Porter Finance. They realized that deploying using clones was 10x cheaper. I suggest applying a similar pattern in factory contracts.

See:

[porter-finance/v1-core#15](https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516)

[(comment) porter-finance/v1-core#34](https://github.com/porter-finance/v1-core/pull/34)

## [G-02] Do-While loops are cheaper than for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times; ) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}

```

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

## [G-03] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are [Solmate](https://github.com/transmissions11/solmate) and [Solady](https://github.com/Vectorized/solady).

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

```solidity
File: contracts/ERC20MultiDelegate.sol


6   import "@openzeppelin/contracts/access/Ownable.sol";
7   import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
8   import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
9   import "@openzeppelin/contracts/utils/math/Math.sol";
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L5-L9

## [G-04] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity

/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(
                    0x40,
                    0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000
                ) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

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

## [G-05] Use assembly to perform efficient back-to-back calls

Use assembly to reuse memory space when making more than one external call.
An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity

contract Called {
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
}

contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns (uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}

contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns (uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(
                gas(),
                calledAddress,
                0x00,
                0x44,
                0x60,
                0x20
            )
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}

```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File: contracts/ERC20MultiDelegate.sol

17        _token.approve(msg.sender, type(uint256).max);
19        _token.delegate(_delegate);
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17-L18

## [G-06] Missing zero address checks in the constructor

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

## [G-07] Use selfdestruct in the constructor if the contract is one-time use

Sometimes, contracts are used to deploy several contracts in one transaction, which necessitates doing it in the constructor.

If the contract’s only use is the code in the constructor, then selfdestructing at the end of the operation will save gas.

Although selfdestruct is set for removal in an upcoming hardfork, it will still be supported in the constructor per [EIP 6780](https://eips.ethereum.org/EIPS/eip-6780).

```solidity
File: contracts/ERC20MultiDelegate.sol

16    constructor(ERC20Votes _token, address _delegate) {
17        _token.approve(msg.sender, type(uint256).max);
18        _token.delegate(_delegate);
19    }




44    constructor(
45        ERC20Votes _token,
46        string memory _metadata_uri
47    ) ERC1155(_metadata_uri) {
48        token = _token;
49    }
```

## [G-08] Use transfer hooks for tokens instead of initiating a transfer from the destination smart contract

Let’s say you have contract A which accepts token B (an NFT or an ERC1363 token). The naive workflow is as follows:

msg.sender approves contract A to accept token B

msg.sender calls contract A to transfer tokens from msg.sender to A

Contract A then calls token B to do the transfer

Token B does the transfer, and calls onTokenReceived() in contract A

Contract A returns a value from onTokenReceived() to token B

Token B returns execution to contract A

This is very inefficient. It’s better for msg.sender to call contract B to do a transfer which calls the tokenReceived hook in contract A.

Note that:

All ERC1155 tokens include a transfer hook

safeTransfer and safeMint in ERC721 have a transfer hook

ERC1363 has transferAndCall

ERC777 has a transfer hook but has been deprecated. Use ERC1363 or ERC1155 instead if you need fungible tokens

If you need to pass arguments to contract A, simply use the data field and parse that in contract A.

```solidity
File: contracts/ERC20MultiDelegate.sol


148        token.transferFrom(proxyAddressFrom, msg.sender, amount);

160        token.transferFrom(msg.sender, proxyAddress, amount);

170        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L148

## [G-09] Use ERC2930 access list transactions when making cross-contract calls to pre-warm storage slots and contract addresses

Access list transactions allow you to prepay the gas costs for some storage and call operations, with a 200 gas discount. This can save gas on further state or storage access, which is paid as a warm access.

If your transaction will make a cross-contract call, you should almost certainly be using access list transactions.

When calling clones or proxies which always involve a cross-contract call via delegatecall, you should make the transaction an access list transaction.

```solidity
File: contracts/ERC20MultiDelegate.sol

147        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
148        token.transferFrom(proxyAddressFrom, msg.sender, amount);



159        address proxyAddress = deployProxyDelegatorIfNeeded(target);
160        token.transferFrom(msg.sender, proxyAddress, amount);



168        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
169        address proxyAddressTo = retrieveProxyContractAddress(token, to);
170        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L147C1-L148C66

## [G-10] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here’s an example of how you can use a hardcoded address instead of address(this):

```solidity
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}

```

In the above example, we have a contract (MyContract) with a public address variable myAddress. Instead of using address(this) to retrieve the contract’s address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[Reference](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: contracts/ERC20MultiDelegate.sol

209                address(this),

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209

## [G-11] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

It's recommended to use the bytes.concat() function instead of abi.encodePacked() for concatenating byte arrays. bytes.concat() is a more gas-efficient and safer way to concatenate byte arrays, and it's considered a best practice in newer Solidity versions.

```solidity
File: contracts/ERC20MultiDelegate.sol

202        bytes memory bytecode = abi.encodePacked(

207            abi.encodePacked(
```

## [G-12] Simple checks for zero can be done using assembly to save gas

```solidity
File: contracts/ERC20MultiDelegate.sol

185        if (bytecodeSize == 0) {

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L185

## [G-13] Do not reduce approval on transferFrom if current allowance is type(uint256).max

[Openzeppelin/ERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/token/ERC20/ERC20.sol#L336)

[OpenZeppelin/openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3085)

```solidity
File: contracts/ERC20MultiDelegate.sol

148        token.transferFrom(proxyAddressFrom, msg.sender, amount);


160        token.transferFrom(msg.sender, proxyAddress, amount);


170        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L148

## [G-14] Avoid having token balances go to zero, always keep a small amount

This is related to the avoiding zero writes section above, but it’s worth calling out separately because the implementation is a bit subtle.

If an address is frequently emptying (and reloading) it’s account balance, this will lead to a lot of zero to one writes.

```solidity
File: contracts/ERC20MultiDelegate.sol

195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L195

## [G-15] Use voting delegation as a gas saving measure

Instead of every token owner voting, only the delegates vote, which net reduces the number of voting transactions.

This is a good [Resource for more](https://www.rareskills.io/post/erc20-votes-erc5805-and-erc6372).

```solidity
File: contracts/ERC20MultiDelegate.sol

8   import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L8

## [G-16] Don’t make variables public unless it is necessary to do so

A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.

```solidity
File: contracts/ERC20MultiDelegate.sol

28    ERC20Votes public token;
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L28

## [G-17] Use branchless algorithms as a replacement for conditionals and loops

The max code from an earlier section is an example of a branchless algorithm, i.e. it eliminates the JUMP opcode, which is more costly than arithmetic opcodes in general.

For loops have jumps built into them, so you may want to consider loop unrolling to save gas.

Loops don’t have to be unrolled all the way. For example, you can execute a loop two items at a time and cut the number of jumps in half.

This is a very extreme optimization, but you should be aware that conditional jumps and loops introduce a slightly more expensive opcode.

```solidity
File: contracts/ERC20MultiDelegate.sol


98            if (transferIndex < Math.min(sourcesLength, targetsLength)) {

101            } else if (transferIndex < sourcesLength) {

104            } else if (transferIndex < targetsLength) {

110        if (sourcesLength > 0) {

113        if (targetsLength > 0) {


185        if (bytecodeSize == 0) {


85        for (

```

## [G-18] Always use Named Returns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}

```

```solidity
File: contracts/ERC20MultiDelegate.sol

189        return proxyAddress;


195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));


214        return address(uint160(uint256(hash)));
```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L189

## [G-19] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of `type(uintX).max`:

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
File: contracts/ERC20MultiDelegate.sol

17        _token.approve(msg.sender, type(uint256).max);

```

## [G-20] Short-circuit booleans

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
