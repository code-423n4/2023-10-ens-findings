# [L-01] Users can create multiple erc-1155 tokenIds for the same delegate
```
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
Because _delegateMulti casts sources and targets from uint256 -> uint160 -> address, it is possible to mint tokens with id > 2^160 - 1, which would represent the same delegatee.

These tokens are possible to transfer, as they are ERC1155 tokens. But it would not be possible to use them to transfer voting power between delegatees, because _processDelegation works only for tokenIds < 2^160.

Such user that entered wrong address as a target, unintentionally or maliciously, will only harm himself, and only in the way that they may not be able to transfer voting power between delegatees.

The affected user will just have to call delegateMulti with `source = tokenId` and empty destination, which will successfully withdraw their tokens.
```
    function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        /*...*/
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } 
            /*...*/

```
```
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source); // checks for tokenId % 2**160
```

## Recommended Mitigation

Use SafeCast for downcasting uint256 to uint160 to protect users from incorrect inputs.