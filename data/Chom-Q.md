## Missing interface IERC20MultiDelegate

There should be an IERC20MultiDelegate interface that developer can use to integrate with their smart contract without having to import the entire contract.

## Missing specific ERC165 supportsInterface

ERC20MultiDelegate should override ERC165 supportsInterface to support IERC20MultiDelegate interface outlined above.

## Approval may be depleted if delegating and undelegating repeatedly

```solidity
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}
```

Only `type(uint256).max` tokens are approved to ERC20MultiDelegate, so if we delegate and undelegated so many times it may be depleted causing subsequent undelegation to be reverted.