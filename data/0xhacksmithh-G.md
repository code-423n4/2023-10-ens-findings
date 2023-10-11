### Gas Optimization Issues
|Title|Instances|Total Gas Saved|Issue|
|-|:-|:-:|:-:|
|[G-1] _delegateMulti() could be more optimizable, if we cache `Math.max` and `Math.min` in local variable | 3 | 650 |
|[G-2] `memory` variable should created outside of the loop |  |  |
|[G-3] Creation of `source` and `target` will be more gas efficient if we don't go for `transferIndex` check with `sourcesLength` and `targetsLength` respectively |  |  |
|[G-4] Reorder Inheritance order so that `ERC1155` state variable could packed with `ERC20MultiDelegate` contract |  |  |
|[G-5] Use of `assert` key word should replace with `require/revert` or `custom errors` |  |  |
|[G-6] Unnessary type casting |  |  |


Total:  6 issues

#

## _delegateMulti() could be more optimizable, if we cache `Math.max` and `Math.min` in local variable
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: approx 650


### Description
`Math.max(sourcesLength, targetsLength)` used to calculate Maximum value from sourcesLength, targetsLength and same for `Math.min`. But these values are calculated multiple times in that function where it could cached in memory.

```solidity
require(
+       uint256 maxVal = Math.max(sourcesLength, targetsLength);
+       uint256 minVal = Math.min(sourcesLength, targetsLength);

-           Math.max(sourcesLength, targetsLength) == amountsLength,
+           maxVal == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-           transferIndex < Math.max(sourcesLength, targetsLength);
+           transferIndex < maxVal;
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

-           if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+           if (transferIndex < minVal) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
```

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L71-L101](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L71-L101)



## `memory` variable should created outside of the loop
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: approx 


### Description
Variable should always created outside of loop then with each iteration of loop that variable get overridden.
By doing so we save memory variable creation cost that that used in every iteration of loop in previous case.

```diff
+       address source;
+       address target;
+       uint256 amount;

        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
-           address source = transferIndex < sourcesLength
+           source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
-           address target = transferIndex < targetsLength
+           target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
-           uint256 amount = amounts[transferIndex];
+           amount = amounts[transferIndex];

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

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90-L96](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90-L96)




## Creation of `source` and `target` will be more gas efficient if we don't go for `transferIndex` check with `sourcesLength` and `targetsLength` respectively
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: approx 


### Description
Instead of checking `transferindex` with `sourceLength` we can directly query for `sources[transferIndex]`.
It will return `0` in case `transferIndex > sourcesLength` and a value in other case.
Then we type cast both to `uint160`

Point to be note here that here omiting `<` gas cost but we adding `uint160` type cast to every situation.

On my testing this method was more gas efficient, some edge cases it will consume more gas,
But overall i think this is more efficient method. 

```solidity
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
```

```diff
+           address source = address(uint160(sources[transferIndex]));
+           address target = address(uint160(targets[transferIndex]));

```

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90-L95](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90-L95)


## Reorder Inheritance order so that `ERC1155` state variable could packed with `ERC20MultiDelegate` contract
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 2.1k


### Description
`ERC1155` have a state variable `string private _uri;`
As we know `string` is a dynamic datatype

and `ERC20MultiDelegate` has one state variable `ERC20Votes public token;` which consume 20bytes

if `_uri` is of small length (i.e < 12bytes) on that case both `_uri` and `token` could stored in one slot, and a extra slot memory will saved

```diff
- contract ERC20MultiDelegate is ERC1155, Ownable {
+ contract ERC20MultiDelegate is Ownable, ERC1155 {
      using Address for address;

      ERC20Votes public token;

```

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L25](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L25)


## Use of `assert` key word should replace with `require/revert` or `custom errors`
- Severity: Gas Optimization
- Confidence: Medium



### Description
On assert failure no gas will refund to caller where gas will refuned on failure of require / revert

```solidity
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);
```

```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

-       assert(amount <= balance);
+       require(amount <= balance, "error");
```

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131)



## Unnessary type casting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: approx 


### Description
No need to typecast uint256(0), instead 0 directly could used.

```diff
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
-               uint256(0), // salt
+               uint256(0), // salt
                keccak256(bytecode)
            )
        );
```

[https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L210](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L210)

