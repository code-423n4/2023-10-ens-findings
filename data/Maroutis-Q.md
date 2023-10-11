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