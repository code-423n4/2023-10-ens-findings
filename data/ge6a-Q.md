## The function delegateMulti is lacking the abiliy to delegate funds from approved accounts

### Description

It often happens in practice that one account is approved from one or several accounts to spend their tokens. In such a case, this account cannot delegate the approved tokens from those accounts using the delegateMulti (https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65-L116) function with a single call. It is not clear from the function's parameters and comments that this is not possible, and a user may try to achieve this by including the approved accounts into the sources array. Such a transaction will likely fail at line 131 (https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131) due to insufficient balance in the ERC1155 contract. Testing this will result in the user losing funds due to gas fees. The way for the described scenario to work is to first transfer the tokens to one account and then call delegateMulti. This solution is not only inefficient in terms of gas costs but also undermines the presence of delegateMulti for this specific situation.

### POC

```
it('delegateMulti is lacking the abiliy to delegate funds from approved accounts', async () => {
    const [owner, tedi, nasko] = await ethers.getSigners();

    let amount = await token.balanceOf(deployer);
    await token.approve(tedi.address, amount);

    await multiDelegate.connect(tedi).delegateMulti([deployer], [nasko.address], [amount]);
 }),
```

In this POC i show that delegateMulti doesn't work for the described scenario and the test above will revert.

### Impact
Unnecessary user gas expenses

### Mitigation steps
- Expanding the capabilities of delegateMulti or adding a new function to cover this scenario.

- In the event that it is decided not to implement such a capability, it is a good idea to explicitly mention that the function does not work under the circumstances described above.