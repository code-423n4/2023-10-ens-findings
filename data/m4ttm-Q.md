# L1 - Use require or custom errors instead of assert
Assert is used in [ERC20MultiDelegate.sol#L131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131) to check that the amount of tokens to transfer (which originates from a user input) is less than or equal to the balance of the source address.

The Solidity documentation says:

“Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.”

whereas

“The require function either creates an error without any data or an error of type Error(string). It should be used to ensure valid conditions that cannot be detected until execution time. This includes conditions on inputs or return values from calls to external contracts.”

Recommend using require or reverting with custom errors instead of using assert, preferably custom errors as they use less gas.

# L2 - Use addresses as inputs instead of uints
In [ERC20MultiDelegate.sol#L58-L60](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L58-L60) `delegateMulti` takes arrays of uints as inputs for `sources` and `targets`, which represent the delegate addresses. The address values are used as ERC1155 IDs which is a uint but the address values themselves are required for other purposes. Using arrays of addresses as arguments instead of uints would be clearer from a user perspective and would remove the issue of double typing a character and delegating to the wrong address by mistake.