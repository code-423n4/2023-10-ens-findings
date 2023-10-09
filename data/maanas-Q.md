### Summary

|ID|Title|Instances|
|-|:-|:-:|
| [NC-01](#nc-01-emit-events-on-important-state-changes)| Emit events on important state changes | 1 |

Total: 1 instances over 1 issues

## Issues

### [NC-01] Emit events on important state changes
The _reimburse function doesn't emit any event. An added event would be helpful for offchain integrations to keep track.
*There are 1 instances of this issue*

```solidity
File: contracts/ERC20MultiDelegate.sol

    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

```

*GitHub*: [144](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144-L149)