## [L01]Tokens sent to the zero Address could be lost

When a user interacts with ERC20MultiDelegate::delegateMulti function, since delegateMulti function takes in addresses as uint256 values, if a user mistakenly inputs 0 amongs the targets, the delegation succeeds.
While this is a user error, if unexpected, the user can lose his/her tokens, without understanding what went wrong, for a technical user, it's still possible to transfer these tokens back, by adding 0(address zero) amongst the source

## [L02]Single-step process for critical ownership transfer

The contract ERC20MultiDelegate inherits from an old version of Openzeppelin that uses a single step ownership transfer process:
The impact is that if an incorrect address, e.g. for which the private key is not known, is used accidentally, it prevents the use of the onlyOwner() functions forever.

RECOMMENDATION
Retain the deployer ownership in the constructor and then use a two-step address change to _governance address separately using setter functions: 1) Approve a new address as a pendingOwner 2) A transaction from the pendingOwner address claims the pending ownership change. This mitigates risk because if an incorrect address is used in step (1) then it can be fixed by re-approving the correct address. Only after a correct address is used in step (1) can step (2) happen and complete the address/ownership change