
# Low Risk and Non-Critical Issues

# [L-01] source address not check for msg.sender 
Source address in _reimburse function is missing check that msg.sender does not transfer the amount to himself.

Link to the code:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L147

Recommendation is made to add check for msg.sender.
```
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
+	require(source != msg.sender, ‘sender cannot be source’);
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```


# [L-02] Amount should be less or equal to transfer index
amount in _reimburse function is missing check to ensure that the amount transferred is less or equal to approved amount.

Link to the code:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L103
