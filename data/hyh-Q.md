**[Q-01] Rounding to uint160 performed for user inputs isn't fully consistent, so ERC20MultiDelegate token ids and events can be made not exactly corresponding to delegate address**

`delegateMulti()` accepts `uint256[]` parameters and operates truncated addresses, e.g. `address(uint160(targets[transferIndex]))`, while receipt minting and burning are performed for original not truncated argument arrays, `sources` and `targets`. This allows for issuing ids that are bigger than delegate address, and those ids will be posted on-chain via `_burnBatch()` and `_mintBatch()` events.

## Impact

Events will be issued with actual delegate addresses being preceded with any random numbers the users might add in excess to uint160, e.g. all `1 << k + address` will be projected to the same address for all `k > 160` due to truncation.

Also, for such `delegateMulti()` usages only direct and reverse delegation will be possible, while `_processDelegation()` will revert as there balance of the truncated id is checked. 

Per high probability and low impact placing severity to be low.

## Proof of Concept

`_delegateMulti()` operates truncated addresses `source` and `target` and untruncated uint256[] `sources` and `targets`:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65-L116

```solidity
    function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        ...

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
>>              ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
>>              ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }

        if (sourcesLength > 0) {
>>          _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
>>          _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }
```

Minting and burning events will be issued for the untruncated `sources[i]` and `targets[j]`, being `ids` in ERC1155 logic:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.3/contracts/token/ERC1155/ERC1155.sol#L306

```solidity
    emit TransferBatch(operator, address(0), to, ids, amounts);
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.3/contracts/token/ERC1155/ERC1155.sol#L369

```solidity
    emit TransferBatch(operator, account, address(0), ids, amounts);
```

## Recommended Mitigation Steps

Checking for the uint160 overflow for consistency, as `_delegateMulti()` calls are relatively rare and adding a small gas cost can be justified in this case, e.g.:

```diff
        } else if (transferIndex < sourcesLength) {
            // Handle any remaining source amounts after the transfer process.
+           require(sources[transferIndex] < type(uint160).max, "Delegate: Source address is too long");
            _reimburse(source, amount);
        } else if (transferIndex < targetsLength) {
+           require(targets[transferIndex] < type(uint160).max, "Delegate: Target address is too long");
            // Handle any remaining target amounts after the transfer process.
            createProxyDelegatorAndTransfer(target, amount);
        }
```

**[Q-02] URI can be set by an owner to a string of an arbitrary length**

Setting URI that is too big can interfere with users holding ERC20MultiDelegate ERC1155 receipts, representing claims on `token` balance of the delegation proxy.

## Proof of Concept

Owner can set an arbitrary big URI:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151-L153

```solidity
    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }
```

## Recommended Mitigation Steps

Consider implementing checks for the `uri` provided, e.g.:

```solidity
    if (bytes(uri).length > 7000) revert URITooLong();
```

On chains guarantees are important and in this case worth a small additional gas cost.