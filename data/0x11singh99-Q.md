# LOW FINDINGS

## [L-01] Floating pragma should not be used

If you leave a floating pragma in your code, you won't know which version was deployed to compile your code, leading to unexpected behavior. If it is compiled with >=0.8.20 then this version introduced PUSH0 opcode. PUSH0 opcode may not yet be implemented on all evm-chains or Layer2s, so deployment on these chains will fail.

```solidity
File : contracts/ERC20MultiDelegate.sol

2: pragma solidity ^0.8.2;

```

[2](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L2)

# NON-CRITICAL ISSUES

## [N-01] Use `_delegatee` instaed of `delegate` for clarity to show that it is referring to whom the voting power/tokens are being delegated to.

```solidity
File : contracts/ERC20MultiDelegate.sol

16: constructor(ERC20Votes _token, address _delegate) {
18:   _token.delegate(_delegate);

```

[16](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L16), [18](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L18)

## [N-02] Do not use param names of function same as inherited state variable name. It will shadow the state variable and create confusion.

```solidity
File : contracts/ERC20MultiDelegate.sol

//@audit uri is defined state variable inside ERC1155 which is being inherited by ERC20MultiDelegate.sol
151: function setUri(string memory uri) external onlyOwner {
152:     _setURI(uri);
153:  }

```

[151-153](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151C5-L153C6)

## [N-03] Solidity layout not followed

/ Layout of Contract: According to solidity Docs
// version
// imports
// errors
// interfaces, libraries, contracts
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private
// internal & private view & pure functions
// external & public view & pure functions
https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

External functions should be defined above internal and private functions but this below function is defined after an internal function which is not proper layout according to solidity docs

```solidity
File : contracts/ERC20MultiDelegate.sol

151: function setUri(string memory uri) external onlyOwner {
152:     _setURI(uri);
153:  }

```

[151-153](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151C5-L153C6)

## [N-04] Use existing `toContract()` function of Address library to check contract existence instead of writing same logic here, which is already implemented in Address library.

It will improve readability and increase modularity.

```solidity
File : contracts/ERC20MultiDelegate.sol

176:  address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
188:     }


```

[176-188](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L176C9-L188C10)

```diff
File : contracts/ERC20MultiDelegate.sol

176:  address proxyAddress = retrieveProxyContractAddress(token, delegate);

-       // check if the proxy contract has already been deployed
-        uint bytecodeSize;
-        assembly {
-            bytecodeSize := extcodesize(proxyAddress)
-        }
-
-        // if the proxy contract has not been deployed, deploy it
-        if (bytecodeSize == 0) {
+         if(!proxyAddress.isContract()){
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
188:     }


```

## [N-05] Unnecessary making a call external to self while it can be called simply because the function is implemented in inherited ERC1155 contract.

This `balanceOf()` function is defined in ERC1155. So calling by directly will give same result in current contract's deployed instance.

```solidity
File : contracts/ERC20MultiDelegate.sol

//@audit this balanceOf function is defined in ERC1155 inherited by current contract so by default it belong to current contract and can be called directly

195: return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));

```

[195](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L195)

```diff
File : contracts/ERC20MultiDelegate.sol

- 195: return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+ 195: return balanceOf(msg.sender, uint256(uint160(delegate)));

```
