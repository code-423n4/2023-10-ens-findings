**Notes**: Each of these optimizations are tested separately with each other. Each optimization is tested against the original code's hardhat gas report.

For the sake of simplicity, only in-scope function `ERC20MultiDelegate.delegateMulti()`'s new gas report (per optimization) will be shown. The other function, `ERC20MultiDelegate.setUri()`, while in scope, is an admin-only function that sets only one storage variable, thus gas optimization is not likely to be impactful.

Original gas report:

```sh
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                6 gwei/gas                ·       1581.28 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      90194  ·      880257  ·     520486  ·           19  ·       4.94  │
·······················|···················|·············|··············|·············|···············|··············
```

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| G-01 | Use Solmate's ERC1155 implementation for gas efficiency | 1 | 825 |
| G-02 | Caching function results within `for` loops will save gas | 1 | 124 |
| G-03 | In the `for` loop, `Math.max(sourcesLength, targetsLength)` can be replaced with `amountsLength` | 1 | 308 |
| G-04 | `sourcesLength > 0 \|\| targetsLength > 0` can be simplified to `amountsLength > 0` | 1 | 24 |
| G-05 | Condition check is not needed in the final `else` block inside the for loop | 1 | 44 |

Totals to **1325** gas saved

## [G-01] Use Solmate's ERC1155 implementation for gas efficiency

[Rari capital's Solmate](https://github.com/transmissions11/solmate) library has implementations for minimalism and gas efficiency. Using their [ERC1155](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC1155.sol) implementation will save gas.

The caveat is that the URI is not available as-is on Solmate's implementation, and such a function `uri()` must be overridden, or functions `_burnBatch()` and `_mintBatch()` has to be renamed to `_batchBurn()` and `_batchMint()`. However that should not be difficult.

Full list of changes:
```solidity=
import "@rari-capital/solmate/src/tokens/ERC1155.sol";
// ...
    string private _uri;
    function _setURI(string memory uri) internal {
        _uri = uri;
    }
    function uri(uint256 /*id*/) public view virtual override returns (string memory) {
        return _uri;
    }
// ... 
    constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) {
        _setURI(_metadata_uri);
        token = _token;
    }
// ...
        if (sourcesLength > 0) {
            _batchBurn(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _batchMint(msg.sender, targets, amounts[:targetsLength], "");
        }
// ... 
```

Saves **825** average gas on local testing. Further gas can be saved by applying techniques given in the bot report (e.g. using custom errors over Solmate's ERC1155 itself).

```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                6 gwei/gas                ·       1590.56 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      89681  ·      879482  ·     519661  ·           19  ·       4.96  │
·······················|···················|·············|··············|·············|···············|··············
```

## [G-02] Caching function results within `for` loops will save gas

In the following `for` loop within `_delegateMulti()`, `Math.max()` and `Math.min()` are called in each iteration.

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85

```solidity=
for (
    uint transferIndex = 0;
    transferIndex < Math.max(sourcesLength, targetsLength);
    transferIndex++
) {
    // ...
    if (transferIndex < Math.min(sourcesLength, targetsLength)) {
        // ...
    }
    // ...
}
```

Simply caching the value in memory before running the loop will save gas. 

*Judging notes*: The `Math.max()` instance was caught by the bot race (finding **[G-21]**), but `Math.min()` was not.

Saves **124** average gas on local testing, **by optimizing `Math.min()` only**. Note that we provide a *different* way to optimize gas on `Math.max()` in **[G-03]** of this report.

```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                5 gwei/gas                ·       1578.48 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      90135  ·      880054  ·     520362  ·           19  ·       4.11  │
·······················|···················|·············|··············|·············|···············|··············
```

## [G-03] In the `for` loop, `Math.max(sourcesLength, targetsLength)` can be replaced with `amountsLength`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87

In line 80, the following condition has already been checked

```solidity
Math.max(sourcesLength, targetsLength) == amountsLength,
```

Therefore, in the for loop in line 87, one can replace `Math.max(sourcesLength, targetsLength)` into `amountsLength` as follow:

```solidity
for (
    uint transferIndex = 0;
    transferIndex < amountsLength; // @audit change like this
    transferIndex++
) 
```

This will eliminate the need to call `Math.max()`. Saves **308** average gas on local testing.

This finding elevates **[G-21]** of the bot report, but saves ~20 more gas by not requiring function result caching, taking advantage of the existing code logic.

```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                6 gwei/gas                ·       1589.69 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      89939  ·      879832  ·     520178  ·           19  ·       4.96  │
·······················|···················|·············|··············|·············|···············|··············
```

## [G-04] `sourcesLength > 0 || targetsLength > 0` can be simplified to `amountsLength > 0`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L75

In line 80, the following condition will be checked

```solidity
Math.max(sourcesLength, targetsLength) == amountsLength,
```

Thus, in the condition in line 80, it is enough to check `amountsLength`, as opposed to either of the other lengths. This is because if `amountsLength` is non-zero, then it is guaranteed that either of the two other arrays is non-empty.

```solidity
require(
    amountsLength > 0, // @audit this much is enough
    "Delegate: You should provide at least one source or one target delegate"
);
```

Saves one condition check, for **24** average gas on local testing.

```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                6 gwei/gas                ·       1589.81 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      90166  ·      880229  ·     520462  ·           19  ·       4.96  │
·······················|···················|·············|··············|·············|···············|··············
```

## [G-05] Condition check is not needed in the final `else` block inside the for loop

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L104

The delegation processing block consists of three branches, each defined by an `if else` statement. The final `else` statement can be used for free without the condition check.

```solidity
if (transferIndex < Math.min(sourcesLength, targetsLength)) {
    // Process the delegation transfer between the current source and target delegate pair.
    _processDelegation(source, target, amount);
} else if (transferIndex < sourcesLength) {
    // Handle any remaining source amounts after the transfer process.
    _reimburse(source, amount);
} else /*if (transferIndex < targetsLength)*/ { // @audit if reached, this is guaranteed to be true
    // Handle any remaining target amounts after the transfer process.
    createProxyDelegatorAndTransfer(target, amount);
}
```

This is because it's logically guaranteed that at least one of these conditions must be true. Thus if the first two blocks are not executed, the final block will always be. Then the final check is redundant.

To ensure clarity, the condition can simply be commented out as shown, instead of being removed.

Saves **44** average gas on local testing. Should be more impactful when `targets` array is much larger than `sources`.

```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·                6 gwei/gas                ·       1589.19 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      90142  ·      880153  ·     520442  ·           19  ·       4.96  │
·······················|···················|·············|··············|·············|···············|··············
```

