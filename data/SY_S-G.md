## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | abi.encode() is less efficient than  abi.encodepacked() | |1| | 
| [G-02] |Use assembly to validate msg.sender | |5| | 
| [G-03] | Use hardcode address instead address(this) | |1| | 
| [G-04] | No need to evaluate all expressions to know if one of them is true | |1| | 
| [G-05] |Can make the variable outside the loop to save gas | |1| | 
| [G-06] |Low level call can be optimized with assembly | |1| | 
| [G-07] |Make 3 event parameters indexed when possible  | |1| | 
| [G-08] | Not using the named return variable when a function returns, wastes deployment gas  | |1| | 
| [G-09] |The result of function calls should be cached rather than re-calling the function | |1| | 
| [G-10] |Non efficient zero initialization | |1| | 
| [G-11] |Use constants instead of type(uintx).max | |1| | 




## Gas Optimizations  
## [G-1] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/ERC20MultiDelegate.sol

204            abi.encode(_token, _delegate)

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L204
## [G-2] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /contracts/ERC20MultiDelegate.sol

17        _token.approve(msg.sender, type(uint256).max);

111            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);

148        token.transferFrom(proxyAddressFrom, msg.sender, amount);

160        token.transferFrom(msg.sender, proxyAddress, amount);

195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17
## [G-3]  Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity 
file: /contracts/ERC20MultiDelegate.sol

209                address(this),

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209
## [G-4] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
file: /contracts/ERC20MultiDelegate.sol

74        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
77        );

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74-L77
## [G-5] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/ERC20MultiDelegate.sol

96            uint256 amount = amounts[transferIndex];

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L96
## [G-6] Low level call can be optimized with assembly     

When using low-level calls, the returnData is copied to memory even if the variable is not utilized. The proper way to handle this is through a low level assembly call. For example:

```solidity
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

can be optimized to:
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}

```


```solidity
file: /contracts/ERC20MultiDelegate.sol

195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L195
## [G-7] Make 3 event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /contracts/ERC20MultiDelegate.sol

33    event DelegationProcessed(
        address indexed from,
        address indexed to,
        uint256 amount
37    );

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L33-L3
## [G-8] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /contracts/ERC20MultiDelegate.sol

///@audit the ' address ' type is declared in function return, in here should not mention if possible.
 
214        return address(uint160(uint256(hash)));

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L214
## [G-9] The result of function calls should be cached rather than re-calling the function

The instances below point to the second+ call of the function within a single function

```solidity
file: /contracts/ERC20MultiDelegate.sol

///@audit the ' retrieveProxyContractAddress ' function is called tow time in one function.

168        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
169        address proxyAddressTo = retrieveProxyContractAddress(token, to);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L168-L169
## [G-10] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity
file: /contracts/ERC20MultiDelegate.sol

86            uint transferIndex = 0;

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L86
## [G-11] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/ERC20MultiDelegate.sol

17        _token.approve(msg.sender, type(uint256).max);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17


