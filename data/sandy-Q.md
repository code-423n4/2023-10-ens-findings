# Summary

## Low Risk Issues
|Id|Title| Instances
|:--:|:-------|:-----:
|L-01| Lack of events for critical functions | 2


## [L-01] Lack of events for critical functions.
Instances:
1. https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144-L149
2. https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161

The functions ``_reimburse`` and ``createProxyDelegatorAndTransfer`` do not emit any events. Event logs are crucial for off-chain services as they notify external users, such as a listening frontend website or client application, that something has happened on the blockchain. External users and blockchain monitoring systems will not be able to easily detect these critical functions and their state changes without events.

##### Recommendations
Emit events for ``_reimburse`` and ``createProxyDelegatorAndTransfer`` functions.

