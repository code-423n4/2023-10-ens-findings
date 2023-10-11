## [L-01] `assert()` does not provide any information when thrown.

*This finding escalates **[N-09]** of the automated report.*

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131

The following assertion will throw when the usre attempts to re-delegate more than the existing amount. Using `assert` here has the following two issues:
- `assert` does not provide any error message when thrown, while it should have been reported told as an `Insufficient balance` error message or similar.
- Even if the condition passes, the tx is guaranteed to fail at the `_burnBatch` call anyway, as the owner would not have enough ERC1155 to burn.

We suggest changing `assert` to a `require` statement, with a clear error message, or using custom errors.

## [L-02] ERC1155 can be inflated if the token doesn't revert on failed transfer

While the only ERC20 token that the contract interacts with is the designated token with voting capability, if the token does not revert on failed transfer (e.g. transfer exceeds balance), then anyone can infinitely mint the ERC1155 token. This can be extended into a honeypot attack vector that can steal funds from succeeded transfers.

**POC**:
- Alice doesn't own any tokens.
- Alice can mint ERC1155 for some delegate for free by using `delegateMulti`.
- Bob mints ERC1155 for the same delegate, using his tokens.
- Alice burns her ERC1155, stealing Bob's tokens.

Alice can be aware of Bob's delegatee by simply front-running.

We recognize this as a low severity issue because, while the impact is high, we could not identify such a token with both voting capability and doesn't revert on failed transfer. However we do not rule out the possibility that such a token does exist.

## [S-01] Tokenizing vote delegation is a new idea, user should be notified/contract should be documented to prevent transferring away their ERC1155

The `ERC20MultiDelegate` splits delegation by wrapping ENS, and tokenizing it into an ERC1155, with the id being the delegatee. We note some of the behaviors that we, as a user, may not expect it to have:
- In the `ERC20MultiDelegate` ERC1155, transferring the token retains the voting power to the current delegatee.
    - In the vanilla ERC20Votes, transferring away ERC20 tokens also transfers away the voting power i.e. if Alice transfers tokens to Bob, then Alice's delegatee's voting power is transferred to Bob's delegatee.
- ERC1155 is a transferrable token. However, transferring away `ERC20MultiDelegate` is equivalent to transferring away ENS. The token may show up on block explorers such as etherscan, and users may attempt to transfer it away, losing their funds as a result.
    - This can be achieved by social engineering or other non-blockchain attacks.

These are some behaviors of the token that we find, while logically sound, should not be unexpected for a user to deduce at first usage, and users should be made aware of the vote tokenizing mechanism to prevent accidentally losing funds.

## [N-01] Documentation should mention minting of ERC1155 when calling `delegateMulti()`

When `delegateMulti()` is called, ERC1155 tokens may be minted. However, following ERC1155 standards, the recipient must either be an EOA, or a contract implementing `ERC1155Holder`, or tx will revert.

While this should be the correct and intended behavior, the fact that ERC1155 is minted is not documented in the function. This may cause unintended reverts to e.g. multisigs, or other interacting contracts.

## [N-02] `delegateMulti()` should be protected with a reentrancy guard.

The function `delegateMulti()` may mint ERC1155 tokens, which in turn may invoke a `onERC1155Received()` at the recipient (if it is a contract). This opens up potential for reentrancy.

While we evaluated that the function correctly follows the CEI pattern and did not identify a potential attack vector, we do believe a proper reentrancy guard in place is good practice. 

## [N-03] `_reimburse()` docs is slightly inconsistent with its behavior

The docs for function `_reimburse()` mentions

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

However the function always (attempt to) transfers exactly `amount`, which should be specified as a function parameter earlier in the `delegateMulti()` call. This is inconsistent with the documentation

