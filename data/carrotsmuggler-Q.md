# 1. `getBalanceForDelegate` check insufficient

The contract `ERC20MultiDelegate.sol` transfers delegations in the `_processDelegation` function. This function first checks if the user has enough erc1155 token balance assigned to the`source`. The erc1155 tokens are used to represent delegations made by the user.

The delegation balance check is done in the following lines [here](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L192-L196).

```solidity
uint256 balance = getBalanceForDelegate(source);
//...

function getBalanceForDelegate(
    address delegate
) internal view returns (uint256) {
    return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
}
```

The issue is that this balance is only updated after processing every transaction. So if a user processes multiple transactions, the balance check is meaningless since the balance isnt updated between the checks.

# 2. Chained ops not supported

After transferring all delegations, the contract goes through all the sources and burns the relevant number of tokens.

```solidity
if (sourcesLength > 0) {
    _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
}
```

After this step, erc1155 tokens are minted for all the delegation targets. The issue is that if a user submits a chained operation, i.e. they delegate votes fom A to B, and then delegate a part of B's votes to C, the contract execution will revert.

This is because the contract will try to burn the delegation tokens for B before even minting them. So users cannot do such chained delegations with this contract.
