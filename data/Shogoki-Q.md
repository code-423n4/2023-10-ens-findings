# QA Report

## Findings - Severity LOW

### L-001 - ERC1155 Tokens with id higher than type(uint160).max can be minted

The `ERC20MultiDelegate` is an ERC1155 Token, which uses the tokenId to represent the address someone delegated it´s tokens to. 
For the main entracne function `delegateMulti` the decision was made to accept arrays of type `uint256` for the source and target addresses, which will be converted to the respective address before they are passed into the underlying functions like this:

```solidity
address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex])) 
                : address(0);
address target = transferIndex < targetsLength
    ? address(uint160(targets[transferIndex]))
    : address(0);
```

At the end, when the actual ERC1155 Tokens are burned and/or minted this conversion is not done, as the id is stored as an `uint256` anyhow.

```solidity
 if (sourcesLength > 0) {
    _burnBatch(msg.sender, sources, amounts[:sourcesLength]); 
}

if (targetsLength > 0) {
    _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
}
```

However, because of this decision it is possible to pass `uint256` values that are higher than the maximum value of an address (uint160). There is no check for this, and the actual transfers will not fail as the value would be (unsafely) downcasted to a valid address (cutting off the higher bits).
The user will receive ERC1155 Tokens with this higher Id, as for the minting (and burnding) no downcasting will happen. 

With these Tokens the user cannot do any redelegation, as the balance check inside `_processDelegation` will be done on the downcasted value. However it is still possible to initiate an reimbursement with these tokens.

#### Impact

Tokens that do not easily map to an underlying address, and that cannot be redelegated, can be created.

#### Tools used

- manual inspection of the code
- Hardhat

#### Recommendation

Either check if the uint values are inside the valid range or change from an uint256 array to an array of `address`, and cast these to uint256 for the mintin/burning.

### L-002 - Incorrect usage of assert

Inside the `_processDelegation` function the users balance of the according ERC1155 token is checked. If the balance is lower than the amount to transfer, the transaction reverts. For this check and the revert Solidity´s `assert` function is used.

```solidity
assert(amount <= balance);
```

The [Solidity documentation](https://docs.soliditylang.org/en/v0.8.21/control-structures.html#panic-via-assert-and-error-via-require) states in regards to `assert`

```
Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.
```

From this it is clear, that the usage of `assert` should be replaced with a corresponding `require` or `revert` call.

#### Impact

Contract could panic revert on invalid user input (too high amount). 

#### Tools used

- manual inspection of the code

#### Recommendation

Change the `assert` statement to an `require` statement, or an if statement with a corresponding `revert` to throw a custom error.
