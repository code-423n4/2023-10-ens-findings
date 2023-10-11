# Low severity findings

## Low-1
A malicious delegatee can lure other ENSToken holders to transfer tokens to their proxy address which would increase their voting power and since the tokens are locked in the proxy the voting power cannot be decreased.

The proxy only has a constructor that delegates tokens and approves the ```ERC20MultiDelegate.sol``` contract for all of it's tokens. There is still a possibility that a user sends ENSTokens directly to a proxy which would trigget the following function: 
```
function _moveVotingPower(
    address src,
    address dst,
    uint256 amount
) private {
    if (src != dst && amount > 0) {
        if (src != address(0)) {
            (uint256 oldWeight, uint256 newWeight) = _writeCheckpoint(_checkpoints[src], _subtract, amount);
            emit DelegateVotesChanged(src, oldWeight, newWeight);
        }

        if (dst != address(0)) {
            (uint256 oldWeight, uint256 newWeight) = _writeCheckpoint(_checkpoints[dst], _add, amount);
            emit DelegateVotesChanged(dst, oldWeight, newWeight);
        }
    }
}
``` 
This would mean that tokens are permanently locked in the proxy and the votes for delegatee are increased forever.