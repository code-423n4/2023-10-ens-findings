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
Because _delegateMulti casts sources and targets from uint256 -> uint160 -> address, but _mintBatch uses the original uint256s, it is possible to mint up to 2^96 different tokens (all uint256s congruent modulo 2^160), which would represent the same delegatee.

These tokens are possible to transfer, as they are ERC1155 tokens. But it would not be possible to use them to transfer voting power between delegatees, because `_processDelegation` works only for tokenIds < 2^160.

Such user that entered wrong address as a target, unintentionally or maliciously, will only harm himself, and only in the way that they may not be able to transfer voting power between delegatees.

The affected user will just have to call delegateMulti with `source = tokenId` and empty destination, which will successfully withdraw their tokens.

It would be also inconvenient for anyone to query delegators' balances: calling `ERC1155#balanceOf(delegator, (uint256(uint160(delegatee)))` may not show the complete picture.
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

# [N-02] Consider emitting DelegationProcessed event in `_reimburse` and `createProxyDelegatorAndTransfer`
```
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
Event `DelegationProcessed` is emitted whenever tokens are moved from one proxy to another, but not emitted when users reimburse their tokens or delegate their own tokens to a new delegatee.

## Recommendation
```diff
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
+       emit DelegationProcessed(source, msg.sender, amount);
    }
```
```
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
+       emit DelegationProcessed(msg.sender, target, amount);
    }
```
`address(0)` instead of `msg.sender` could also be used, but `msg.sender` would be more informative and convenient as it would show the delegator.

# [N-03] Assertion for tokenId >= 2^160 can be bypassed with a different token 

Same as L-01, this issue arises from unsafe downcasting of uint256 to address. 

### Scenario
1. Alice calls 
```
delegateMulti(
    sources = []
    targets = [bob, bob + 2^160]
    amounts = [1e18, 1e18]
)
```
receives 1e18 tokens with `id == bob + 2^160`
and 1e18 tokens with `id == bob`.
2. Alice calls 
```
delegateMulti(
    sources = [bob + 2^160]
    targets = [charlie]
    amounts = [1e18]
)
```
the flow is the following:
delegateMulti -> _delegateMulti -> _processDelegation
```
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);
```
```
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```
The code above checks if Alice has enough tokens with `id == bob`, not with `id == bob + 2^160`, and does not revert, although Alice is using a different tokenId.

The contract will then successfully burn the token with `id == bob + 2^160` from Alice.

Had Alice not have enough `bob + 2^160` tokens, the assertion would still be passed. But, thanks to `_burnBatch`, the execution would revert as Alice would not have had enough `bob` tokens.

```
_burnBatch(msg.sender, sources, amounts[:sourcesLength]);
```

Although `_burnBatch` serves as a second assertion in the current implementation, there may occur problems if `_delegateMulti` or `_burnBatch` is ever edited.

The recommendation from L-01 is sufficient to prevent this issue, as it would become impossible to mint tokens with id >= 2^160.

