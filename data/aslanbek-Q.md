# [L-01] Users can create multiple erc-1155 tokenId's for the same delegate

```
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```

because delegateMulti casts sources and targets from uint256 -> uint160 -> address, it is possible to create tokens with id > 2^160, which would represent the same user. 2^96 addresses is possible to create

However, such tokens can not be used for transferring voting power between delegatees, as _processDelegation checks mod 2^160 of a token

1. Alice creates id and `id + 2 ^ 161`
2. 