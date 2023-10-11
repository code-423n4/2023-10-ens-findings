## Missing Events for delegation
Events are needed to inform external clients of changes in the smart contract. The `ERC20MultiDelegate`  contract only emits the `DelegationProcessed` events when voting power is redelegated. It should also be emitted or similar events should be emitted when voting power is undelegated and initially delegated. This will allow external clients to keep track of critical changes.

## It's difficult to get all the delegates of a user.
The `ERC20MultiDelegate` contract does not provide a view method to get all the delegates to an address easily. To do this currently, you have to know all the delegates and get the ERC1155 balance the address has for each delegate. A single method to retrieve an array of all the delegates of an address would make things easier. It should also be noted that adding such a method would require the developer to update this array when a transfer of ERC1155 tokens is done.