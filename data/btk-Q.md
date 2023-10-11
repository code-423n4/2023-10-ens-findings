# ENS Overview

ENS is a decentralized naming service built on top of the Ethereum blockchain. It is designed to provide naming and resolution services for a wide range of resources, including blockchain addresses, decentralized content, and user profile information.

**`ERC20MultiDelegate.sol`** is a specific smart contract within the ENS protocol. It serves as a multi-delegation mechanism for ERC20 tokens that support the ERC20Votes extension. This contract allows users to delegate their voting power to multiple addresses in a single transaction.

- *The contract leverages Solidity's create2 opcode for creating proxy contracts.*
- *It does not use custom cryptographic algorithms but relies on the ERC20Votes and ERC1155 standards for managing delegation and token metadata, respectively.*

# User Flow

![Diagram](https://i.redd.it/66wg0nm1ljtb1.png)

| Total Low issues |
|------------------|

| Risk    | Issues Details                                             | Number        |
|---------|------------------------------------------------------------|---------------|
| [QA-01] | Address library is not used anywhere in the code           | 1             |
| [QA-02] | Add a reentrancy guard to the `delegateMulti()` function   | 1             |
| [QA-03] | Add foundry tests                                          |               |
| [QA-04] | Take warnings seriously                                    | 5             |
| [QA-05] | Lack of event emit                                         | 1             |
| [QA-06] | Solidity compiler optimizations can be problematic         | 1             |
| [QA-07] | Use SMTChecker                                             |               |

## [QA-01] Address library is not used anywhere in the code 

#### Description

In the **`ERC20MultiDelegate`** contract, the Address library is imported and used with the using statement, but it serves no purpose and is not utilized anywhere in the code.

#### Lines of code 

```solidity
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
```

- [ERC20MultiDelegate.sol#L4](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L4)

```solidity
    using Address for address;
```

- [ERC20MultiDelegate.sol#L26](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L26)

#### Recommended Mitigation Steps

Remove the unused import and code involving the Address library to keep the codebase clean and efficient.

## [QA-02] Add a reentrancy guard to the `delegateMulti()` function

#### Description

The **`ERC20MultiDelegate`** contract, is an ERC1155, which have multiple hooks that could potentially lead to reentrancy issues. It's important to note that, in this specific context, any reentrant calls by an attacker are mitigated by the fact that the hook is executed after the critical operations, as demonstrated in the following code snippet:

```solidity
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
```

However, for the sake of clarity and to facilitate future development and understanding, it is advisable to implement a proper documentation within the `delegateMulti()` function, especially if other contracts plan to build on top of the **`ERC20MultiDelegate`**.

#### Lines of code 

```solidity
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```

- [ERC20MultiDelegate.sol#L114](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L114)

#### Recommended Mitigation Steps

Document the possibility of reentrancy attacks after the tokens are minted in the `delegateMulti()` function.

## [QA-03] Add foundry tests

#### Description

Foundry lets your write tests in Solidity, versus Hardhat which uses JavaScript, so you don't have to learn a JS library and pay all the associated context switching costs. Foundry also has native fuzzing capabilities to catch edge cases, and is written in Rust so tests run a lot faster. That said, having both hardhat and foundry tests in your repo is a great plus to the codebase.

#### Recommended Mitigation Steps

Add foundry tests. A basic foundry setup that I've written for this contest. You can find it at the following GitHub repository:

> https://github.com/0xbtk/ens-tests

## [QA-04] Take warnings seriously

#### Description

If the compiler warns you about something, you should change it. Even if you do not think that this particular warning has security implications, there might be another issue buried beneath it. Any compiler warning we issue can be silenced by slight changes to the code.

> Reference: https://docs.soliditylang.org/en/v0.8.17/security-considerations.html#take-warnings-seriously

#### Lines of code 

```solidity
    function setUri(string memory uri) external onlyOwner {
```

- [ERC20MultiDelegate.sol#L151](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151)

```solidity
    function createProxyDelegatorAndTransfer(
```

- [ERC20MultiDelegate.sol#L155](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155)

```solidity
    function transferBetweenDelegators(
```

- [ERC20MultiDelegate.sol#L163](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163)

```solidity
    function deployProxyDelegatorIfNeeded(
```

- [ERC20MultiDelegate.sol#L173](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173)

```solidity
    function getBalanceForDelegate(
```

- [ERC20MultiDelegate.sol#L192](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L192)

#### Recommended Mitigation Steps

Fix the compiler warnings.

## [QA-05] Lack of event emit

#### Description

The **`setUri()`** function do not emit an event when the uri changes, something that it's very important for dApps and users. Events help non-contract tools to track changes and prevent users from being surprised by changes.

#### Lines of code 

```solidity
    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }
```

- [ERC20MultiDelegate.sol#L151-L153](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151-L153)

#### Recommended Mitigation Steps

Add Event-Emit

## [QA-06] Solidity compiler optimizations can be problematic

#### Description

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### Lines of code 

```solidity
    optimizer: {
      enabled: true, // runs: 200 by default
    },
```

- [hardhat.config.js#L26-L28C7](https://github.com/code-423n4/2023-10-ens/blob/main/hardhat.config.js#L26-L28C7)

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [QA-07] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Reference: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19   

#### Lines of code 

```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```

- [ERC20MultiDelegate.sol#L57-L63](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57-L63)

#### Recommended Mitigation Steps

Use formal mathematical verification
