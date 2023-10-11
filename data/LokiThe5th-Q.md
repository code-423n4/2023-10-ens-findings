## Low-01: The allowance for a given proxy address will be consumed as time goes on  

The `ERC20MultiDelegate` contract is intended to be used with the `ENSToken`. The deployed `ENSToken` does not implement an infinite approval. Should the allowance be consumed the voting power and the tokens will remain stuck in the `ERC20ProxyDelegator`.

The deployed `ENSToken` can be seen [here](https://etherscan.io/token/0xc18360217d8f7ab5e7c516566761ea12ce7f9d72#code#F2#L149): 

```
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        _transfer(sender, recipient, amount);

        uint256 currentAllowance = _allowances[sender][_msgSender()];
        require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
        unchecked {
            _approve(sender, _msgSender(), currentAllowance - amount);
        }

        return true;
    }
```

## PoC  

Because tokens can be delegated and undelegated at will, and due to the fact that `ENSToken` does not implement "infinite approval" the allowance granted by a given `ERC20ProxyDelegator` will be consumed as delegations from that address are processed. 

This is easily verified by checking the values for `ENSToken::allowance(proxyDelegator, multiDelegate)` before and after the `multiDelegate` tests are run.

It must be noted that the allowance granted, `uint256` is so large that it is unlikely to be exhausted through normal operation and is economically extremely costly to exploit on Ethereum Mainnet with the `ENSToken`. It would be possible in theory for a griefer to do this, but the economic cost would be prohibitive. 

Should the allowance run out, however, then all the tokens in the `ERC20ProxyDelegator` will be stuck.

## Recommendation  
This only affects tokens using `ERC20` versions that do not implement infinite approvals. Mitigating this would require implementing functionality to "top-up" the allowance should it run out. This may not be necessary for `ENSToken` with it's current and future supply, but other projects hoping to use the `ERC20MultiDelegate` contract should keep this in mind.