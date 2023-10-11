## The `ERC20MultiDelegate.delegateMulti` function doesn't check if target or source address is zero values.

The delegator has the ability to delegate tokens to a zero address value. This action serves no practical purpose because the delegate(`address(0)`) will be unable to vote.

### Recommended Mitigation Steps
Consider checking if the source or target addresss aren't `address(0)`.


## The `ERC20MultiDelegate._reimburse` function doesn't check if msg.sender has tokens to reimburse.

The `ERC20MultiDelegate._processDelegation` function includes a check to determine if `msg.sender` has the required balance. It is advisable to consider adding a similar check for reimburse tokens:

```diff
function _reimburse(address source, uint256 amount) internal {
+       uint256 balance = getBalanceForDelegate(source);
+       assert(amount <= balance);
		
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```