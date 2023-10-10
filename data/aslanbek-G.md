# [G-01] Call balanceOf internally 
[ERC20MultiDelegate.sol#L195](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L195) 

```diff
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
-       return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+       return balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```
Before:
```
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      89747  ·      894689  ·     525133  ·           19  ·       4.17  │
·······················|···················|·············|··············|·············|···············|··············
|  Deployments                             ·                                          ·  % of limit   ·             │
···········································|·············|··············|·············|···············|··············
|  ERC20MultiDelegate                      ·          -  ·           -  ·    4261617  ·       14.2 %  ·      33.80  │
···········································|·············|··············|·············|···············|··············

```
After:
```
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      89747  ·      888300  ·     524293  ·           19  ·       4.16  │
·······················|···················|·············|··············|·············|···············|··············
|  Deployments                             ·                                          ·  % of limit   ·             │
···········································|·············|··············|·············|···············|··············
|  ERC20MultiDelegate                      ·          -  ·           -  ·    4211239  ·         14 %  ·      33.41  │
···········································|·············|··············|·············|···············|··············
```
# [G-02] function _delegateMulti - avoid if-else and ternary operator overuse by splitting the loop into smaller ones
[ERC20MultiDelegate.sol#L85-L108](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L108)
Instead of a single "swiss-army-knife" loop that handles every input via if-else and ternary operators, it will be better to separate the loop into smaller ones.

The expected input is: a amounts, s sources, t targets; a = max(s,t). 
```
        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
For `transferIndex` ∈ `[0; min(sourcesLength, targetsLength))`, only function `_processDelegation` is called. So we're moving it into a separate loop, avoiding plenty of branchings:
```
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }
```
For `transferIndex` ∈ `[min(sourcesLength, targetsLength), max(sourcesLength, targetsLength))`, we create a separate loop (one for sources and one for targets):
```
        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```

Original version:

```
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
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```
Optimized version:
```
        uint256 minSourcesTargetsLength = Math.min(
            sourcesLength,
            targetsLength
        );
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }

        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```
The following output is the result of creating 5 delegatees, then moving their voting power to another addresses.

While the optimization makes the deployment ~29k gas more expensive, it saves ~230 gas per iteration, so there only need to be at least ~126 iterations (iteration is 1 transfer of voting power between two accounts) over the contract's lifetime to break even.
```
| contracts/ERC20MultiDelegate.sol:ERC20MultiDelegate contract |                 |        |        |         |         |
|--------------------------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2100669                                                      | 11018           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 214448          | 617632 | 617632 | 1020817 | 2       |
```
```
| contracts/ERC20MultiDelegate.sol:ERC20MultiDelegate contract |                 |        |        |         |         |
|--------------------------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2129498                                                      | 11162           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 213567          | 616474 | 616474 | 1019381 | 2       |
```
[Multidelegate.t.sol](https://gist.github.com/aslanbekaibimov/10135b8d15888a9c2bbaa3814d511bdf)