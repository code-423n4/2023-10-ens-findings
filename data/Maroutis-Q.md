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

