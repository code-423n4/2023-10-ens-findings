# `assert` Function Misuse

## Impact
- Unintended gas consumption, potentially leading to higher transaction costs for users.
- Possible indication of deeper logical issues within the contract, as `assert` is intended to validate invariants.

## Proof of Concept
Utilizing `assert` in Solidity, especially versions prior to 0.8.0, can lead to unintended consequences regarding gas usage. In versions before 0.8.0, a failed `assert` consumes all remaining gas in a transaction. From 0.8.0 onwards, while it doesn’t consume all remaining gas, it does consume a specific amount, designed to discourage its use. The Solidity documentation underscores that a failed `assert` typically indicates a contract bug, as it should not be triggered during normal operation or through external inputs.

There is 1 instance of this issue: 
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131

The Solidity assert() function is meant to assert invariants.
Properly functioning code should never reach a failing assert statement. However, it is clear that in this case, a simple user input mistake can lead to a failing assert.

## Recommended Mitigation Steps
- Opt for `require` instead of `assert` for checks that aren’t validating invariants, and ensure to provide clear error messages to facilitate debugging.


# Any token sent without using the `delegateMulti` function gets permanently stuck

The proxy contract is designed to work with `ERC20Votes` tokens sent using the `delegateMulti`, but it doesn’t prevent any additionnal tokens or any other types of tokens from being sent to it. If a user mistakenly sends tokens to the contract, they could become stuck with no mechanism in place to retrieve them.

Example: 
If a user mistakenly sends standard tokens to the contract, they might expect similar functionality or the ability to retrieve them, which is not supported in the current implementation.

## Impact
- Loss of tokens: Any tokens sent directly to the contract could be irretrievable.

## Recommended Mitigation Steps
- Create a function that allows the `ERC20MultiDelegate` contract owner to retrieve any additional tokens sent to the proxies contracts mistakenly. The additionnal tokens are easily identifiable since they do not have a ERC1155 counterparty. The owner can then transfer the additionnal tokens to the corresponding owners.

# Tokens Not Reverting on Failure

## Impact
- **Unexpected Behavior:** While this is technically compliant with the ERC20 standard, ERC20 Tokens like ZRX and EURS, which return `false` instead of reverting on failure, can lead to unexpected contract behavior if calls to `transferFrom` are not properly checked. Equivalent tokens that implement the ERC20Votes can have the exact behaviors.
- **Security Risks:** Failing to handle the `false` return value could expose the contract to logical errors between the value of ERC1155 tokens minted and the actual tokens sent to the proxy.

## Proof of Concept
1. The contract calls the `transferFrom` function of a token that returns `false` on failure (e.g., ZRX, EURS).
2. The contract does not check the return value of `transferFrom`, assuming it will revert on failure.
3. The contract continues execution despite the failed token transfer, leading to logical errors.

## Tools Used
- Code review.

## Recommended Mitigation Steps
- Add Check for the return value of `transferFrom` and revert or handle failure as needed in https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L148
- Consider using the OpenZeppelin `SafeERC20` library, which converts return values to reverts to ensure consistency.

# Revert on Large Approvals & Transfers

## Impact
- **Incompatibility Issues:** While this is technically compliant with the ERC20 standard, tokens like UNI and COMP revert on large approvals and transfers, which might not be expected by interacting contracts and could cause transactions to fail. Equivalent tokens that implement ERC20Votes might display the same behaviors.
- **Logical Errors:** The `ERC20ProxyDelegator` contract is expecting the maximum approved value to be reflected in the allowances mapping.

## Proof of Concept
1. The contract calls `approve` or `transfer` with a value larger than `uint96` on tokens like UNI or COMP.
2. The token contract reverts, causing the transaction to fail and possibly leading to unexpected behavior in interacting contracts.

## Tools Used
- Code review.

## Recommended Mitigation Steps
- Specify a curated list of whitelisted tokens to use
- Ensure that the contract logic can handle reverts gracefully and provide clear error messages to users.

# Revert on Zero Value Transfers

This is more of a specific issue. A user wishes to create a proxy contract without sending any tokens. He wants to send tokens later on.

## Impact
- **Transaction Failures:** While this is technically compliant with the ERC20 standard, tokens like LEND revert on zero-value transfers, which might cause transactions to fail unexpectedly if not handled properly. Equivalent tokens that implement ERC20Votes might display the same behaviors.
- **User Experience:** Users might encounter failed transactions and be unsure of the cause if zero-value transfers are not handled or communicated effectively.

## Proof of Concept
1. The contract attempts to transfer a zero-value amount of a token like LEND.
2. The token contract reverts, causing the transaction to fail and possibly leading to a poor user experience.

## Tools Used
- Code review.
- Interaction simulations with various ERC20 tokens.

## Recommended Mitigation Steps
- Ensure that the contract logic checks for and prevents zero-value transfers where they are not allowed or expected.
- Specify a curated list of whitelisted tokens to use