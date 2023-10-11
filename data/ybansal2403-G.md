# Summary

Some optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.7`, `optimizer enabled:false`, and `200 runs`. Optimizations that were not benchmarked are explained properly. The following command was used to run the tests for the benchmarks below: `yarn test test/delegatemulti.js`.

## Gas Optimizations

| Number                                                | Issue                                            | Instances |
| ----------------------------------------------------- | :----------------------------------------------- | :-------: |
| [G-01](#use-do-while-loops-instead-of-for-loops)      | Use `do while` loops instead of `for loops`      |     1     |
| [G-02](<#use-hardcode-address-instead-address(this)>) | Use hardcode address instead address(this)       |     1     |
| [G-03](#use-for-unsigned-integer-comparison)          | Use != 0 or == 0 for unsigned integer comparison |     1     |

<a name='use-do-while-loops-instead-of-for-loops'></a>

## [G-01] Use `do while` loops instead of `for loops`

A do while loop will cost less gas since the condition is not being checked for the first iteration. Other optimizations, such as caching length, precrementing, and using unchecked blocks are not included in the Diff since they are included in the bot race report.

Total Instances: `1`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85C9-L108C10

_Gas Savings for `ERC20MultiDelegate.delegateMulti`, obtained via protocol's tests: Avg 162 gas_

|        | Min   | Max    | Avg    | # calls |
| ------ | ----- | ------ | ------ | ------- |
| Before | 90194 | 880257 | 520486 | 19      |
| After  | 90044 | 880069 | 520324 | 19      |

```solidity
File: contracts/ERC20MultiDelegate.sol
85:for (
86:            uint transferIndex = 0;
87:            transferIndex < Math.max(sourcesLength, targetsLength);
88:            transferIndex++
89:        ) {
.            address source = transferIndex < sourcesLength
.                ? address(uint160(sources[transferIndex]))
.                : address(0);
.            address target = transferIndex < targetsLength
105:               // Handle any remaining target amounts after the transfer process.
106:               createProxyDelegatorAndTransfer(target, amount);
107:           }
108:       }

```

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..a25e0c5 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -82,11 +82,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         );

         // Iterate until all source and target delegates have been processed.
-        for (
-            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
-            transferIndex++
-        ) {
+        uint transferIndex;
+        do {
             address source = transferIndex < sourcesLength
                 ? address(uint160(sources[transferIndex]))
                 : address(0);
@@ -105,7 +102,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 // Handle any remaining target amounts after the transfer process.
                 createProxyDelegatorAndTransfer(target, amount);
             }
-        }
+            ++transferIndex;
+        } while (transferIndex < Math.max(sourcesLength, targetsLength));

         if (sourcesLength > 0) {
             _burnBatch(msg.sender, sources, amounts[:sourcesLength]);


```

<a name='use-hardcode-address-instead-address(this)'></a>

## [G-02] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}
```

Total Instances: `1`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L209C16-L209C31

```solidity
File: contracts/ERC20MultiDelegate.sol
209:  address(this),
```

<a name='use-for-unsigned-integer-comparison'></a>

## [G-03] Use != 0 or == 0 for unsigned integer comparison

_The following instance is missed in the automated report_

It's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Total Instances: `1`

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L110C9-L115C10

_Gas Savings for `ERC20MultiDelegate.delegateMulti`, obtained via protocol's tests: Avg 6 gas_

|        | Min   | Max    | Avg    | # calls |
| ------ | ----- | ------ | ------ | ------- |
| Before | 90194 | 880257 | 520486 | 19      |
| After  | 90188 | 880251 | 520480 | 19      |

```solidity
File: contracts/ERC20MultiDelegate.sol
110:if (sourcesLength > 0) {
111:            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
112:        }
113:        if (targetsLength > 0) {
114:            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
115:        }
```

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..eb25109 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -107,10 +107,10 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
             }
         }

-        if (sourcesLength > 0) {
+        if (sourcesLength != 0) {
             _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
         }
-        if (targetsLength > 0) {
+        if (targetsLength != 0) {
             _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
         }
     }

```
## `GasReport` output with all optimizations applied
```js
·------------------------------------------|----------------------------|-------------|-----------------------------·
|           Solc version: 0.8.7            ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
···········································|····························|·············|······························
|  Methods                                 ·               15 gwei/gas                ·       1553.57 usd/eth       │
·······················|···················|·············|··············|·············|···············|··············
|  Contract            ·  Method           ·  Min        ·  Max         ·  Avg        ·  # calls      ·  usd (avg)  │
·······················|···················|·············|··············|·············|···············|··············
|  ENSRegistry         ·  setResolver      ·          -  ·           -  ·      48841  ·           16  ·       1.14  │
·······················|···················|·············|··············|·············|···············|··············
|  ENSRegistry         ·  setSubnodeOwner  ·          -  ·           -  ·      50147  ·           16  ·       1.17  │
·······················|···················|·············|··············|·············|···············|··············
|  ENSToken            ·  approve          ·      26990  ·       46986  ·      45558  ·           14  ·       1.06  │
·······················|···················|·············|··············|·············|···············|··············
|  ENSToken            ·  mint             ·          -  ·           -  ·      79517  ·           16  ·       1.85  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  delegateMulti    ·      90038  ·      880063  ·     520318  ·           19  ·      12.13  │
·······················|···················|·············|··············|·············|···············|··············
|  ERC20MultiDelegate  ·  setUri           ·          -  ·           -  ·      32694  ·            1  ·       0.76  │
·······················|···················|·············|··············|·············|···············|··············
|  Deployments                             ·                                          ·  % of limit   ·             │
···········································|·············|··············|·············|···············|··············
|  ENSRegistry                             ·          -  ·           -  ·    1084520  ·        3.6 %  ·      25.27  │
···········································|·············|··············|·············|···············|··············
|  ENSToken                                ·          -  ·           -  ·    4238796  ·       14.1 %  ·      98.78  │
···········································|·············|··············|·············|···············|··············
|  ERC20MultiDelegate                      ·          -  ·           -  ·    4119553  ·       13.7 %  ·      96.00  │
···········································|·············|··············|·············|···············|··············
|  PublicResolver                          ·          -  ·           -  ·    3584728  ·       11.9 %  ·      83.54  │
·------------------------------------------|-------------|--------------|-------------|---------------|-------------·

```