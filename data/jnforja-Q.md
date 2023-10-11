# Delegating to address zero breaks the invariant multiDelegate.balanceOf(address1, address2) <= ERC20Votes.getVotes(address2)

## Description
`ERC20MultiDelegate.delegateMulti` doesn't revert on the zero address, but the zero address is not considered a valid delegate by the ERC20Votes which leads to situations where after calling `delegateMulti` the value of `ERC20Votes.getVotes(address2)` will be updated but the value of `multiDelegate.balanceOf(address1, address2)` won't.

##POC
Add the following it block to `test/delegatemulti.js` inside of the `describe block` "ENS Multi Delegate"
```javascript

describe("ENS Multi Delegate", () => {
...
   it("delegating to address zero breaks the invariant multiDelegate.balanceOf(address1, address2) <= ERC20Votes.getVotes(address2)", async () => {
      const delegatorTokenAmount = await token.balanceOf(deployer);
      await token.approve(multiDelegate.address, delegatorTokenAmount);

      await multiDelegate.delegateMulti(
        [],
        [ethers.constants.AddressZero],
        [delegatorTokenAmount]
      );

      const delegatorTokenAmountAfter = await token.balanceOf(deployer);
      expect(delegatorTokenAmountAfter.toString()).to.equal("0");

      const votesOfZero = await token.getVotes(ethers.constants.AddressZero);
      expect(votesOfZero.toString()).to.equal("0");
      expect(
        await multiDelegate.balanceOf(deployer, ethers.constants.AddressZero)
      ).to.eq(delegatorTokenAmount);
    });
```
})
## Method
Manual Review

## Fix
Forbid address 0 as a delegatee