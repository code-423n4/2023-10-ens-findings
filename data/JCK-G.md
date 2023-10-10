

# Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Amounts should be checked for 0 before calling a transfer  | 3 |    |
|[G-02]| Use constants instead of type(uintx).max  | 1 |    |
|[G-03]| Use assembly to perform efficient back-to-back calls  | 1 |    |
|[G-04]| Use hardcoded address instead of address(this)  | 3 |    |
|[G-05]| abi.encode() is less efficient than abi.encodepacked()  | 1 | 51317  |
|[G-06]| Counting down in for statements is more gas efficient  | 1 | 12171   |
|[G-07]| Use assembly to check for address(0)  | 2 | 12  |
|[G-08]| Use calldata instead of memory for function arguments that do not get mutated  | 1 |  52482  |
|[G-09]| Don't initialize variables with default value  | 1 |    |
|[G-10]| Loop best practice to save gas  | 1 |    |


## [G-01] Amounts should be checked for 0 before calling a transfer

It can be beneficial to check if an amount is zero before invoking a transfer function, such as transfer or safeTransfer, to avoid unnecessary gas costs associated with executing the transfer operation. If the amount is zero, the transfer operation would have no effect, and performing the check can prevent unnecessary gas consumption.

```solidity
file:   contracts/ERC20MultiDelegate.sol

148     token.transferFrom(proxyAddressFrom, msg.sender, amount);

160     token.transferFrom(msg.sender, proxyAddress, amount);

170     token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L148


## [G-02] Use constants instead of type(uintx).max

using type(uintx).max can result in higher gas costs because it involves a runtime operation to calculate the maximum value at runtime. This calculation is performed every time the expression is evaluated.

To save gas, it is recommended to use constants instead of type(uintx).max to represent the maximum value. By declaring a constant with the maximum value, the value is known at compile-time and does not require any runtime calculations.


```solidity
file:  contracts/ERC20MultiDelegate.sol

17    _token.approve(msg.sender, type(uint256).max);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17


## [G-03] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.



```solidity
file:    contracts/ERC20MultiDelegate.sol

168      address proxyAddressFrom = retrieveProxyContractAddress(token, from);
169      address proxyAddressTo = retrieveProxyContractAddress(token, to);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L168-L169

## [G-04] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file:  contracts/ERC20MultiDelegate.sol


209    address(this),

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209


## [G-05] abi.encode() is less efficient than abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Before using of encodePacked gas value per one abi.encode: 405030
After using of  encodePacked instead of encode gas value per one abi.encodePacked: 353713 

Tools used remix

```solidity
file:  contracts/ERC20MultiDelegate.sol

204    abi.encode(_token, _delegate)

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L204

## [G‑06] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix



```solidity
file:  contracts/ERC20MultiDelegate.sol

85     for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85-89


###  Test they code 

```solidity
file:

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}

```


## [G-07] Use assembly to check for address(0)

  Save 6 gas per instance.

```solidity
file:  contracts/ERC20MultiDelegate.sol

92     : address(0);

95     : address(0);

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L92


## [G-08] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Befor using of calldata gas value:  456573
After using of calldata gas value:  404091

Tools used Remix 

```solidity
file:  contracts/ERC20MultiDelegate.sol

151    function setUri(string memory uri) external onlyOwner {

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151

## [G-09] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.


```solidity
file:  contracts/ERC20MultiDelegate.sol

86     uint transferIndex = 0;

```

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L86


## [G-10] Loop best practice to save gas


```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}

```

```solidity
file:  contracts/ERC20MultiDelegate.sol

85     for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85






