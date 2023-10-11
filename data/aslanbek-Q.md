# [L-01] Users can create multiple ERC-1155 tokenIds for the same delegate
```
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
```
    _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```
Because _delegateMulti casts sources and targets from uint256 -> uint160 -> address, but _mintBatch uses the uint256s, it is possible to mint up to 2^96 different tokens (all uint256s congruent modulo 2^160), which would represent the same delegatee.

These tokens are possible to transfer, as they are ERC1155 tokens. But it would not be possible to use them to transfer voting power between delegatees, because `_processDelegation` works only for tokenIds < 2^160.

Such user that entered wrong address as a target, unintentionally or maliciously, will only harm himself, and only in the way that they may not be able to transfer voting power between delegatees.

The affected user will just have to call delegateMulti with `source = tokenId` and empty destination, which will successfully withdraw their tokens.

It would be also inconvenient for anyone to query delegators' balances: calling `ERC1155#balanceOf(delegator, (uint256(uint160(delegatee)))` will not show the complete picture.
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

Use SafeCast for downcasting uint256 -> uint160 to protect users from their incorrect inputs.

# [N-01] library `Address` is imported but never used
[ERC20MultiDelegate.sol#L4](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4)
```
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
```
[ERC20MultiDelegate.sol#L26](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L26)
```
    using Address for address;
```

# [N-02] function getBalanceForDelegate is redundant
```
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```
`getBalanceForDelegate` has no additional logic in comparison to `balanceOf`. Consider using `balanceOf` directly.

 ```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
-       uint256 balance = getBalanceForDelegate(source);
+       uint256 balance = balanceOf(msg.sender, uint256(uint160(source)));

        // N-09 from the bot report
-       assert(amount <= balance);
+       require(amount <= balance, "Insufficient Balance");

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);


        emit DelegationProcessed(source, target, amount);
    }
```