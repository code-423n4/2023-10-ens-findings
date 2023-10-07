
The insuffecient amount reverts with no clear message to users, line : https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L103


## Proof of Concept 
The reimbursement of remaining source amounts on the delegation process runs into the loop with the given amounts, since the reimbursement runs on a loop, the amounts passed may reverts when they're unsefficient,  the whole delegation process will revert with the basic `ERC20: transfer amount exceeds balance`

users may not know to whome/which balance refers to, it's better adding a custom error that checks the balance of proxyAddressFrom in ENS token against the given amounts, example :

```
 function _reimburse(address source, uint256 amount) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        require(token.balanceOf(proxyAddressFrom) >= amount, "proxyAddressFrom does not have this amount");
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```

PoC :

```
it('Revert with clear error when proxyAddressFrom.balance is not suffecient', async () => {
      const delegatorTokenAmount = await token.balanceOf(deployer);
      await token.approve(multiDelegate.address, delegatorTokenAmount);
      const delegates = [deployer, alice, bob, charlie];
      const amounts = delegates.map(() =>
        delegatorTokenAmount.div(delegates.length)
      );

      await multiDelegate.delegateMulti([], delegates, amounts);

      const wrongDelegates = [deployer, alice, bob, dave];
      const withdrawAmounts = wrongDelegates.map(() =>
        delegatorTokenAmount.div(wrongDelegates.length)
      );

      await expect(
        multiDelegate.delegateMulti(wrongDelegates, [], withdrawAmounts)
      ).to.be.revertedWith('proxyAddressFrom does not have this amount');
    });
```

Log :

```
 npx hardhat test test/delegatemulti.js

  ENS Multi Delegate
    re-deposit
      ✔ Revert with clear error when proxyAddressFrom.balance is not suffecient
```