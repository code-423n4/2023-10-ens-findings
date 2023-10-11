## Gas Optimizations

| Number                                                                         | Issue                                                          | Instances |
| ------------------------------------------------------------------------------ | :------------------------------------------------------------- | :-------: |
| [[G-01](#g-01-use-constants-instead-of-typeuintxmax)]                          | Use constants instead of type(uintX).max                       |     1     |
| [[G-02](#g-02-cache-calculation-rather-than-re-calculating-on-each-iteration)] | Cache calculation rather than re-calculating on each iteration |     5     |
| [[G-03](#g-03-custom-error-cost-less-than-requireassert)]                      | Custom `error` cost less than `require`/`assert                |     1     |
| [[G-04](#g-04-keccak256-hash-of-literals-should-only-be-computed-once)]        | `keccak256()` hash of literals should only be computed once    |     1     |
| [[G-05](#g-05-use-hardcode-address-instead-addressthis)]                       | Use hardcode address instead address(this)                     |     1     |
| [[G-06](#g-06-dont-cache-value-if-it-is-only-used-once)]                       | `Don’t cache` value if it is only used once                    |     1     |
| [[G-07](#g-07-do-not-initialize-state-variables-with-their-default-value)]     | Do not initialize state variables with their `default value`   |     1     |

## [G-01] Use constants instead of type(uintX).max

In Solidity, type(uintX).max is used to represent the maximum value for an unsigned integer of X bits. This value is often used in contracts to represent an "infinity" value or indicate that a certain condition has not been met. However, using type(uintX).max can be expensive in terms of gas cost since it requires the compiler to perform a complex computation to determine the maximum value for an unsigned integer of X bits.

**_1 Instance_**

```solidity

File : contracts/ERC20MultiDelegate.sol

15:  contract ERC20ProxyDelegator {
       constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
     }
20:   }

```

[15-20](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L15C1-L20C2)

```diff
File : contracts/ERC20MultiDelegate.sol

  contract ERC20ProxyDelegator {

+   uint256 constant MAX_UINT256 = 2**256 - 1;
       constructor(ERC20Votes _token, address _delegate) {
-       _token.approve(msg.sender, type(uint256).max);
+       _token.approve(msg.sender, MAX_UINT256);
        _token.delegate(_delegate);
     }
   }

```

## [G-02] Cache calculation rather than re-calculating on each iteration

**_5 Instances_**

```solidity

File : contracts/ERC20MultiDelegate.sol

79:               require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
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

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
116:    }

```

[79-116](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79C1-L116C6)

```diff
File : contracts/ERC20MultiDelegate.sol

+       uint256 MAX_length = Math.max(sourcesLength, targetsLength);
+       uint256 MIN_length = Math.min(sourcesLength, targetsLength);
        require(
-            Math.max(sourcesLength, targetsLength) == amountsLength,
+            MAX_length == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < MAX_length;
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

-            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
+            if (transferIndex <MIN_length) {
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
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
116:    }

```

## [G-03] Custom `error` cost less than `require`/`assert`

Consider the use of a custom `error`, as it leads to a cheaper deploy cost and run time cost. The run time cost is only relevant when the revert condition is met.

_Note: These instances missed by bot report_

**_1 Instance_**

```solidity

File : contracts/ERC20MultiDelegate.sol

131:  assert(amount <= balance);

```

[131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131)

## [G-04] `keccak256()` hash of literals should only be computed once

The result of the hash should be stored in an immutable variable, and the variable should be used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

**_1 Instance_**

```solidity

File : contracts/ERC20MultiDelegate.sol

206:    bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
213:    );

```

[206-213](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L206C8-L213C11)

## [G-05] Use hardcode address instead address(this).

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. In Solidity, the address(this) expression returns the address of the current contract instance. This expression is commonly used to send or receive Ether to or from the contract.
Using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.

**_1 Instance_**

```solidity

File :

209:  address(this),

```

[209](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209C15-L209C31)

## [G-06] `Don’t cache` value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

**_1 Instance_**

```solidity

File : contracts/ERC20MultiDelegate.sol

124:   function _processDelegation(
           address source,
           address target,
           uint256 amount
        ) internal {
           uint256 balance = getBalanceForDelegate(source);

           assert(amount <= balance);

           deployProxyDelegatorIfNeeded(target);
           transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
137:    }

```

[124-137](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L124C4-L137C6)

```diff
File : contracts/ERC20MultiDelegate.sol

   function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
-       uint256 balance = getBalanceForDelegate(source);

-       assert(amount <= balance);
+       assert(amount <= getBalanceForDelegate(source));

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

    emit DelegationProcessed(source, target, amount);
    }

```

## [G-07] Do not initialize state variables with their `default value`

Every variable assignment in Solidity costs gas. When initializing variables, we often waste gas by assigning default values that will never be used.

**_1 Instance_**

```solidity

File : contracts/ERC20MultiDelegate.sol

85:   for (
              uint transferIndex = 0;
              transferIndex < Math.max(sourcesLength, targetsLength);
              transferIndex++
89:        ) {

```

[85-89](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85C6-L89C12)

```diff
File : contracts/ERC20MultiDelegate.sol

   for (
-         uint transferIndex = 0;
+         uint transferIndex;
          transferIndex < Math.max(sourcesLength, targetsLength);
          transferIndex++
       ) {

```
