# Gas Optimizations Report

| Gas Optimizations | Issues                                                                                        | Instances |
|-------------------|-----------------------------------------------------------------------------------------------|-----------|
| [G-01]            | Unnecessary function can be removed                                                           | 1         |
| [G-02]            | Use do-while loop instead of for-loop to save gas                                             | 1         |
| [G-03]            | Remove initialization to 0 to save gas                                                        | 1         |
| [G-04]            | Remove parameter `ERC20Votes _token` from function `retrieveProxyContractAddress` to save gas | 1         |
| [G-05]            | Use if conditional statements instead of ternary operators to save gas                        | 1         |
| [G-06]            | Use gas-efficient assembly for common math operations like min and max                        | 3         |
| [G-07]            | Consider using alternatives to OpenZeppelin                                                   | 1         |

### Total number of issues: 9 instances across 7 issues

### Total deployment gas saved: 35209 gas

### Total function execution gas saved: 488 gas (per call)

### Total gas saved: 35697 gas

## [G-01] Unnecessary function can be removed

Although following the pattern of external functions calling internal functions is recommended, the [delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6) function just calls internal [_delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65) function with the passed parameters. This is an unnecessary step. Additionally, there are no complex interactions when calling from the external to internal function, thus making it safe to remove.

There is 1 instance of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6

**Before VS After**

**Deployment cost: 4121917 - 4116745 = 5172 gas saved**

**Function execution cost: 90194 - 90140 = 54 gas saved per call**

Remove this external [delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6) function below and make the internal function [_delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65) to external.
```solidity
File: ERC20MultiDelegate.sol
60:     function delegateMulti(
61:         uint256[] calldata sources,
62:         uint256[] calldata targets,
63:         uint256[] calldata amounts
64:     ) external {
65:         _delegateMulti(sources, targets, amounts);
66:     }
```

## [G-02] Use do-while loop instead of for-loop to save gas

There is 1 instance of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85C1-L108C10

**Before VS After**

**Deployment cost: 4121917 - 4118209 = 3708 gas saved**

**Function execution cost: 90194 - 89798 = 396 gas saved per call**

Instead of this:
```solidity
File: ERC20MultiDelegate.sol
089:         for (
090:             uint transferIndex = 0;//@audit Gas - check if initialization to 0 can be removed to save gas
091:             transferIndex < Math.max(sourcesLength, targetsLength);
092:             transferIndex++ //@audit Gas - ++i
093:         ) {
094:             address source = transferIndex < sourcesLength
095:                 ? address(uint160(sources[transferIndex]))
096:                 : address(0);
097:             address target = transferIndex < targetsLength
098:                 ? address(uint160(targets[transferIndex]))
099:                 : address(0);
100:             uint256 amount = amounts[transferIndex];
101:     
102:             if (transferIndex < Math.min(sourcesLength, targetsLength)) {
103:                 // Process the delegation transfer between the current source and target delegate pair.
104:                 _processDelegation(source, target, amount);
105:             } else if (transferIndex < sourcesLength) {
106:                 // Handle any remaining source amounts after the transfer process.
107:                 _reimburse(source, amount);
108:             } else if (transferIndex < targetsLength) {
109:                 // Handle any remaining target amounts after the transfer process.
110:                 createProxyDelegatorAndTransfer(target, amount);
111:             }
112:         }
```
Use this:
```solidity
File: ERC20MultiDelegate.sol
113:         uint256 transferIndex;
114:         do {
115:             address source = transferIndex < sourcesLength
116:                 ? address(uint160(sources[transferIndex]))
117:                 : address(0);
118:             address target = transferIndex < targetsLength
119:                 ? address(uint160(targets[transferIndex]))
120:                 : address(0);
121:             uint256 amount = amounts[transferIndex];
122:     
123:             if (transferIndex < Math.min(sourcesLength, targetsLength)) {
124:                 // Process the delegation transfer between the current source and target delegate pair.
125:                 _processDelegation(source, target, amount);
126:             } else if (transferIndex < sourcesLength) {
127:                 // Handle any remaining source amounts after the transfer process.
128:                 _reimburse(source, amount);
129:             } else if (transferIndex < targetsLength) {
130:                 // Handle any remaining target amounts after the transfer process.
131:                 createProxyDelegatorAndTransfer(target, amount);
132:             }
133: 
134:             unchecked {
135:                 ++transferIndex;
136:             }
137:         } while (transferIndex < Math.max(sourcesLength, targetsLength));
```

## [G-03] Remove initialization to 0 to save gas

Since unsigned integer variables are 0 by default, there is no need to initialize them to 0.

There is 1 instance of this issue:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L86C32-L86C35

**Before VS After**

**Deployment cost: 4121917 - 4121905 = 12 gas saved**

```solidity
86: uint transferIndex = 0;
```

## [G-04] Remove parameter `ERC20Votes _token` from function `retrieveProxyContractAddress` to save gas

The function `retrieveProxyContractAddress` is called by functions `_reimburse`, `transferBetweenDelegators` and `deployProxyDelegatorIfNeeded`. These functions always pass the state variable `token` as parameter to the `retrieveProxyContractAddress`. This creates extra gas that can be avoided by just using the state variable `token` in the `retrieveProxyContractAddress` function directly.

There is 1 instance of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L198

**Before VS After**

**Deployment cost: 4121917 - 4098816 = 23101 gas saved**

**Function execution cost: 90194 - 90184 = 10 gas saved per call** 

Instead of this:
```solidity
File: ERC20MultiDelegate.sol
232:     function retrieveProxyContractAddress(
233:         ERC20Votes _token,
234:         address _delegate
235:     ) private view returns (address) {
236:         bytes memory bytecode = abi.encodePacked(
237:             type(ERC20ProxyDelegator).creationCode, 
238:             abi.encode(_token, _delegate)
239:         );
240:         bytes32 hash = keccak256(
241:             abi.encodePacked(
242:                 bytes1(0xff),
243:                 address(this),               
244:                 uint256(0), // salt
245:                 keccak256(bytecode)
246:             )
247:         );
248:         return address(uint160(uint256(hash)));
249:     }
```
Use this: **(Note: Make sure to make changes in functions `_reimburse`, `transferBetweenDelegators` and `deployProxyDelegatorIfNeeded` when passing parameters to `retrieveProxyContractAddress`)**
```solidity
File: ERC20MultiDelegate.sol
232:     function retrieveProxyContractAddress(
233:         address _delegate //@audit Now only one parameter
234:     ) private view returns (address) {
235:         bytes memory bytecode = abi.encodePacked(
236:             type(ERC20ProxyDelegator).creationCode, 
237:             abi.encode(token, _delegate) //@audit directly using state variable "token"
238:         );
239:         bytes32 hash = keccak256(
240:             abi.encodePacked(
241:                 bytes1(0xff),
242:                 address(this),               
243:                 uint256(0), // salt
244:                 keccak256(bytecode)
245:             )
246:         );
247:         return address(uint160(uint256(hash)));
248:     }
```

## [G-05] Use if conditional statements instead of ternary operators to save gas

There is 1 instance of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L90C1-L96C53

**Before VS After**

**Deployment cost: 4121917 - 4118701 = 3216 gas saved**

**Function execution cost: 90194 - 90166 = 28 gas saved per call**

Instead of this:
```solidity
File: ERC20MultiDelegate.sol
94:             address source = transferIndex < sourcesLength
95:                 ? address(uint160(sources[transferIndex]))
96:                 : address(0);
97:             address target = transferIndex < targetsLength
98:                 ? address(uint160(targets[transferIndex]))
99:                 : address(0); 
```
Use this:
```solidity
File: ERC20MultiDelegate.sol
100:             address source;
101:             address target;
102:             if (transferIndex < sourcesLength) source = address(uint160(sources[transferIndex]));
103:             if (transferIndex < targetsLength) target = address(uint160(targets[transferIndex]));
```

## [G-06] Use gas-efficient assembly for common math operations like min and max

There are 3 instances of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L80
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98C39-L98C39

These are the following min and max instances in the code.
```solidity
File: contracts/ERC20MultiDelegate.sol
80: Math.max(sourcesLength, targetsLength) == amountsLength,
87: transferIndex < Math.max(sourcesLength, targetsLength);
98: if (transferIndex < Math.min(sourcesLength, targetsLength)) {
```
The current Math library being used by OpenZeppelin evaluates "max" using ternary operators as follows:
```solidity
    function max(uint256 a, uint256 b) internal pure returns (uint256) {
        return a > b ? a : b;
    }
```
An optimized version for this in assembly would be as follows (Similar to [Solady's implementation](https://github.com/Vectorized/solady/blob/08901b1131be4903e989884571198d72de1c0ee9/src/utils/FixedPointMathLib.sol#L693)):
```solidity
    assembly {
        c := xor(a, mul(xor(a, b), gt(b, a)))
    }
```
The reason the above example is more gas efficient is because the ternary operator in the original code contains conditional jumps in the opcodes, which are more costly.

## [G-07] Consider using alternatives to OpenZeppelin

Consider using [Solmate](https://github.com/transmissions11/solmate) and [Solady](https://github.com/Vectorized/solady). Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly. In the [ERC20MultiDelegate.sol](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) contract, ERC1155 is used frequently for batch minting and burning. Using a gas-optimized version can help reduce gas costs on the user end.

There is 1 instance of this:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4C1-L9C54

```solidity
File: ERC20MultiDelegate.sol
4: import {Address} from "@openzeppelin/contracts/utils/Address.sol";
5: 
6: import "@openzeppelin/contracts/access/Ownable.sol"; 
7: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
8: import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
9: import "@openzeppelin/contracts/utils/math/Math.sol";
```