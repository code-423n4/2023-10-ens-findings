## [01] Redundant Import and Usage of OpenZeppelin's Address library.

OpenZeppelin's [https://docs.openzeppelin.com/contracts/4.x/api/utils#Address](Address) library includes various functions related to the `address` type. 

**ERC20MultiDelegate.sol** imports the library and uses it on the `address` type here:

```solidity
using Address for address;
```

This is completely redundant though as the contract never actually uses any of the libraries functions. 


## [02] Redundant casting to address(0)

In the following part of the code there are 2 instances of redundant casting to `address(0)`.

```solidity
address source = transferIndex < sourcesLength
    ? address(uint160(sources[transferIndex]))
    : address(0);
address target = transferIndex < targetsLength
    ? address(uint160(targets[transferIndex]))
    : address(0);
    uint256 amount = amounts[transferIndex];
```

This code is inside the for-loop which will continue executing as long as `transferIndex` is less than the maximum value between `sourcesLength` and `targetsLength`. This basically casts the `uint256` value of the `transferIndex`-nth of the array to an address and assigns it to source / target as long as`transferIndex` is <  `sourcesLength` / `targetsLength`. If its not source / target is assigned `address(0)`. 

Assigning `address(0)` to source / target is completely unnecessary though as the variable will never get used in that case.

Because whenever one of the two values ends up being address(0) the following **if / else if** checks end up calling a function where that value is never used.

If source is `address(0)` because of `transferIndex` >=  `sourcesLength` we end up calling:

```solidity
else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        createProxyDelegatorAndTransfer(target, amount);
}
```

and if target is `address(0)` because of `transferIndex` >= `targetsLength` we end up calling:

```solidity
else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        _reimburse(source, amount);
} 
```