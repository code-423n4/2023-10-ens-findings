## Non Critical Issues

|       | Issue                                                           |
| ----- | :-------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                            |
| NC-2  | BE EXPLICIT DECLARING TYPES                                     |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                        |
| NC-4  | COMMENTED CODE                                                  |
| NC-5  | CONTRACT CHECK                                                  |
| NC-6  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION   |
| NC-7  | MISSING CHECKS FOR `ADDRESS(0)`                                 |
| NC-8  | MISSING EVENTS FOR CRITICAL PARAMETER/STATE CHANGE              |
| NC-9  | NO SAME VALUE INPUT CONTROL                                     |
| NC-10 | USE A MORE RECENT VERSION OF SOLIDITY                           |
| NC-11 | UNUSED RETURN VALUES                                            |
| NC-12 | LACK OF CHECKS ON INTEGER RANGES IN ERC20MULTIDELEGATE CONTRACT |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

151:     function setUri(string memory uri) external onlyOwner {

152:         _setURI(uri);

```

### [NC-2] BE EXPLICIT DECLARING TYPES

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

86:             uint transferIndex = 0;

179:         uint bytecodeSize;

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] COMMENTED CODE

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

210:                 uint256(0), // salt

```

### [NC-5] CONTRACT CHECK

#### Description:

Checking if a call was made from an Externally Owned Account (EOA) or a contract account is typically done using `extcodesize` check which may be circumvented by a contract during construction when it does not have source code available. Checking if `tx.origin == msg.sender` is another option. Both have implications that need to be considered.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

181:             bytecodeSize := extcodesize(proxyAddress)

```

### [NC-6] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

2: pragma solidity ^0.8.2;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-7] MISSING CHECKS FOR `ADDRESS(0)`

#### Description:

The affected code in the provided smart contract is the lack of checks for `address(0)` in functions like `transferFrom()` and other similar token-related functions. Specifically, it occurs in functions where addresses are used without validation, potentially leading to unintended transfers and unauthorized state modifications.

#### **Proof Of Concept**

```solidity
function _reimburse(address source, uint256 amount) internal {
    // Transfer the remaining source amount or the full source amount
    // (if no remaining amount) to the delegator
    address proxyAddressFrom = retrieveProxyContractAddress(token, source);
    token.transferFrom(proxyAddressFrom, msg.sender, amount);
}

```

In this function, `proxyAddressFrom` is retrieved without checking if it's `address(0)`, allowing the `transferFrom()` operation to be executed with the zero address as the source. This could lead to unexpected transfers to `address(0)`.

#### Recommended Mitigation Steps:

To address this issue, you should add a validation check to ensure that `proxyAddressFrom` is not `address(0)` before proceeding with the `transferFrom()` operation. This validation step will prevent unintended transfers to the zero address.

Add code like this: `if (proxyAddressFrom == address(0)) revert ADDRESS_ZERO();` or `require(proxyAddressFrom != address(0), "Address cannot be zero");`

### [NC-8] MISSING EVENTS FOR CRITICAL PARAMETER/STATE CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

Events for critical state changes (e.g. owner and other critical parameters) should be emitted for tracking this off-chain.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

151:     function setUri(string memory uri) external onlyOwner {

```

### [NC-9] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

48:         token = _token;

```

### [NC-10] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

2: pragma solidity ^0.8.2;

```

### [NC-11] UNUSED RETURN VALUES

In the ERC20MultiDelegate contract, there are several function calls that return values, but these return values are not used in the contract. Unused return values of function calls can indicate potential programmer errors and might lead to unexpected behavior. It's essential to handle the return values appropriately to ensure the correct execution flow of the contract.

In the ERC20ProxyDelegator constructor, the `_token.approve` function returns a boolean value indicating the success of the approval, but this return value is not used.

In the `_processDelegation `function, the `ERC1155(this).balanceOf` function returns the balance of the source delegate, but the return value is not used for any validation or further processing.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

15: contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}

192: function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }

```

#### Recommended Mitigation Steps:

Review the functions with unused return values and ensure that these return values are appropriately handled. Depending on the use case, unused return values might indicate a logic error or unnecessary function calls.

### [NC-12] LACK OF CHECKS ON INTEGER RANGES

The contract lacks checks on the integer arguments passed to various functions. Proper checks on integer ranges are crucial to prevent unexpected behavior due to overflow or underflow conditions. While the contract performs some validations, such as ensuring the number of amounts matches the number of sources and targets, it does not validate the ranges of the integer inputs. Depending on the use case, it might be necessary to add explicit checks for valid integer ranges to avoid potential vulnerabilities.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

15: contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}

124: function _processDelegation(
    address source,
    address target,
    uint256 amount
) internal {
    uint256 balance = getBalanceForDelegate(source); // Unused return value

    assert(amount <= balance);

    deployProxyDelegatorIfNeeded(target);
    transferBetweenDelegators(source, target, amount);

    emit DelegationProcessed(source, target, amount);
}

```

#### Recommended Mitigation Steps:

Implement explicit checks on integer ranges wherever applicable. Validate input parameters to prevent overflow, underflow, or unintended values that could lead to unexpected behavior.

## Low Issues

|     | Issue                                                          |
| --- | :------------------------------------------------------------- |
| L-1 | BAD NOMENCLATURE                                               |
| L-2 | IPFS HASH DECODING VULNERABILITY                               |
| L-3 | OWNER CAN RENOUNCE OWNERSHIP                                   |
| L-4 | NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES |

### [L-1] BAD NOMENCLATURE

#### Description:

The interface `IERC20` contains two methdos that are not pressent in the official ERC20, `delegate`, it’s recommended to change the name of the contract because not any ERC20 it’s valid.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

18:         _token.delegate(_delegate);

```

### [L-2] IPFS HASH DECODING VULNERABILITY

#### Description:

The ERC20MultiDelegate contract is susceptible to an IPFS hash decoding vulnerability. The contract currently assumes a fixed IPFS hash format, with a hardcoded hash function identifier (0x12 for SHA-256) and hash length (0x20 for 32 bytes). This rigid approach may lead to compatibility issues if IPFS adopts different hash functions or hash lengths in the future. Consequently, the contract might misinterpret IPFS hashes, potentially compromising the integrity and functionality of IPFS-based features.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

208:                 bytes1(0xff),

```

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

###

### [L-3] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

6: import "@openzeppelin/contracts/access/Ownable.sol";

25: contract ERC20MultiDelegate is ERC1155, Ownable {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-4] NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES

#### Description:

The package.json configuration file says that the project is using 4.3.1 of OpenZeppelin which has a not last update version.

#### **Proof Of Concept**

```solidity
File: package.json

14:    "@openzeppelin/contracts": "^4.3.1",

```

#### Recommended Mitigation Steps:

Use patched versions.
