## Summary table

|     Index     | Issue                                            | Instances | Deployment | Avg. Method |
| :-----------: | :----------------------------------------------- | :-------: | :--------: | :---------: |
| [G-01](#G-01) | [Disallow Transfer of Zero Tokens](#G-01)        |     3     |     -      |      -      |
| [G-02](#G-02) | [Unnecessary max Length Lookup.](#G-02)          |     1     |   2 352    |     308     |
| [G-03](#G-03) | [Redundant function](#G-03)                      |     1     |   5 184    |     54      |
| [G-04](#G-04) | [Do not cache variable for only one use](#G-04)  |     5     |   8 793    |     150     |
| [G-05](#G-05) | [Do not compute condition in every loop.](#G-05) |     1     |   −1268    |     214     |

11 instances over 5 issues

---

Initial Gas:

|      Contract      |    Method     |  Min  |   Max   |    Avg     | # calls | usd (avg) |
| :----------------: | :-----------: | :---: | :-----: | :--------: | :-----: | :-------: |
| ERC20MultiDelegate | delegateMulti | 90194 | 880257  |   520486   |   19    |   5.75    |
| ERC20MultiDelegate |    setUri     |   -   |    -    |   32694    |    1    |   0.36    |
|    Deployments     |               |       |         | % of limit |         |           |
| ERC20MultiDelegate |       -       |   -   | 4121917 |   13.7 %   |  45.50  |           |

---

### [G-01] Disallow Transfer of Zero Tokens

<a name="G-01"></a>
[To the top](#TOP)

Transferring zero token amount serves no purpose, consumes user gas needlessly, and generates insignificant events that may lead to confusion among users and developers.

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 148](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L148), [Source: ERC20MultiDelegate.sol, lines 160](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L1160), [Source: ERC20MultiDelegate.sol, lines 170](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L170)

<details>

<summary>see 3 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
Line 148: 148        token.transferFrom(proxyAddressFrom, msg.sender, amount);
Line 160: 160        token.transferFrom(msg.sender, proxyAddress, amount);
Line 170: 170        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
```

</details>

#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..5f6c215 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -36,6 +36,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256 amount
     );

+    error ZeroValueTransfer();
+
     /**
      * @dev Constructor.
      * @param _token The ERC20 token address
@@ -95,6 +97,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 : address(0);
             uint256 amount = amounts[transferIndex];

+            if (amount == 0) revert ZeroValueTransfer();
+
             if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                 // Process the delegation transfer between the current source and target delegate pair.
                 _processDelegation(source, target, amount);
```

---

### [G-02] Unnecessary max Length Lookup.

<a name="G-02"></a>
[To the top](#TOP)

There is a check that  so there is not need to lookup max length in further code

There is a check that compares the maximum length between sources and targets with amounts length `Math.max(sourcesLength, targetsLength) == amountsLength`. Since the maximum length is already determined, there is no need to perform additional length lookups in the subsequent code. This redundancy can lead to inefficiency and potential security risks if not properly addressed.

By eliminating the unnecessary length lookup, we can improve the code's performance and reduce the chance of introducing vulnerabilities.

Deployment: **2 352**

Average Method call: **308**

|      Contract      |    Method     |  Min  |   Max   |  Avg   | # calls | usd (avg) |
| :----------------: | :-----------: | :---: | :-----: | :----: | :-----: | :-------: |
| ERC20MultiDelegate | delegateMulti | 89939 | 879832  | 520178 |   19    |   5.73    |
| ERC20MultiDelegate |    setUri     |   -   |    -    | 32694  |    1    |   0.36    |
|    Deployments     |               |       |         |        |         |           |
| ERC20MultiDelegate |       -       |   -   | 4119565 | 13.7 % |  45.40  |           |


#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 79-89](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L79-89)

<details>

<summary>see 1 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
79         require(
80             Math.max(sourcesLength, targetsLength) == amountsLength,
81             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82         );
83 
84         // Iterate until all source and target delegates have been processed.
85         for (
86             uint transferIndex = 0;
87             transferIndex < Math.max(sourcesLength, targetsLength);
88             transferIndex++
89         ) {
```

</details>


#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..f2839b8 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -84,7 +84,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         // Iterate until all source and target delegates have been processed.
         for (
             uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < amountsLength;
             transferIndex++
         ) {
             address source = transferIndex < sourcesLength
```

---

### [G-03] Redundant function

<a name="G-03"></a>
[To the top](#TOP)

The `delegateMulti` function currently serves as a redundant intermediate step, as it simply calls the `_delegateMulti` function without any additional logic. To optimize performance and eliminate unnecessary jumps, it is recommended to merge these two functions into a single streamlined implementation. This consolidation will lead to more efficient code execution.

Deployement: **5 184**

Average Method call: **54**

|      Contract      |    Method     |  Min  |   Max   |    Avg     | # calls | usd (avg) |
| :----------------: | :-----------: | :---: | :-----: | :--------: | :-----: | :-------: |
| ERC20MultiDelegate | delegateMulti | 90140 | 880203  |   520432   |   19    |   5.75    |
| ERC20MultiDelegate |    setUri     |   -   |    -    |   32694    |    1    |   0.36    |
|    Deployments     |               |       |         | % of limit |         |           |
| ERC20MultiDelegate |       -       |   -   | 4116733 |   13.7 %   |  45.48  |           |

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 57-63](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-63)

<details>

<summary>see 1 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
57    function delegateMulti(
58        uint256[] calldata sources,
59        uint256[] calldata targets,
60        uint256[] calldata amounts
61    ) external {
62        _delegateMulti(sources, targets, amounts);
63    }
```

</details>

#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..f4183b7 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -59,14 +59,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256[] calldata targets,
         uint256[] calldata amounts
     ) external {
-        _delegateMulti(sources, targets, amounts);
-    }
-
-    function _delegateMulti(
-        uint256[] calldata sources,
-        uint256[] calldata targets,
-        uint256[] calldata amounts
-    ) internal {
         uint256 sourcesLength = sources.length;
         uint256 targetsLength = targets.length;
         uint256 amountsLength = amounts.length;
```

---

### [G-04] Do not cache variable for only one use

<a name="G-04"></a>
[To the top](#TOP)

Do not cache variable for only one use

Deployment: **8 793**

Average Method Call: **150**

|      Contract      |    Method     |  Min  |   Max   |    Avg     | # calls | usd (avg) |
| :----------------: | :-----------: | :---: | :-----: | :--------: | :-----: | :-------: |
| ERC20MultiDelegate | delegateMulti | 90116 | 880101  |   520336   |   19    |   5.74    |
| ERC20MultiDelegate |    setUri     |   -   |    -    |   32694    |    1    |   0.36    |
|    Deployments     |               |       |         | % of limit |         |           |
| ERC20MultiDelegate |       -       |   -   | 4113124 |   13.7 %   |  45.35  |           |

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 131](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L131), [Source: ERC20MultiDelegate.sol, lines 148](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L148), [Source: ERC20MultiDelegate.sol, lines 160](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160), [Source: ERC20MultiDelegate.sol, lines 170](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L170), [Source: ERC20MultiDelegate.sol, lines 203](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L203), 

<details>

<summary>see 5 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
131        assert(amount <= balance);
...
148        token.transferFrom(proxyAddressFrom, msg.sender, amount);
...
160        token.transferFrom(msg.sender, proxyAddress, amount);
...
170        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
...
203        type(ERC20ProxyDelegator).creationCode, 
```

</details>

#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..f6a0511 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -126,9 +126,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address target,
         uint256 amount
     ) internal {
-        uint256 balance = getBalanceForDelegate(source);
-
-        assert(amount <= balance);
+        assert(amount <= getBalanceForDelegate(source));
 
         deployProxyDelegatorIfNeeded(target);
         transferBetweenDelegators(source, target, amount);
@@ -144,8 +142,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
     function _reimburse(address source, uint256 amount) internal {
         // Transfer the remaining source amount or the full source amount
         // (if no remaining amount) to the delegator
-        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
-        token.transferFrom(proxyAddressFrom, msg.sender, amount);
+        token.transferFrom(retrieveProxyContractAddress(token, source), msg.sender, amount);
     }
 
     function setUri(string memory uri) external onlyOwner {
@@ -156,8 +153,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address target,
         uint256 amount
     ) internal {
-        address proxyAddress = deployProxyDelegatorIfNeeded(target);
-        token.transferFrom(msg.sender, proxyAddress, amount);
+        token.transferFrom(msg.sender, deployProxyDelegatorIfNeeded(target), amount);
     }
 
     function transferBetweenDelegators(
@@ -165,9 +161,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address to,
         uint256 amount
     ) internal {
-        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-        address proxyAddressTo = retrieveProxyContractAddress(token, to);
-        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
+        token.transferFrom(retrieveProxyContractAddress(token, from), retrieveProxyContractAddress(token, to), amount);
     }
 
     function deployProxyDelegatorIfNeeded(
@@ -199,18 +193,16 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         ERC20Votes _token,
         address _delegate
     ) private view returns (address) {
-        bytes memory bytecode = abi.encodePacked(
-            type(ERC20ProxyDelegator).creationCode, 
-            abi.encode(_token, _delegate)
-        );
-        bytes32 hash = keccak256(
+        return address(uint160(uint256(keccak256(
             abi.encodePacked(
                 bytes1(0xff),
                 address(this),
                 uint256(0), // salt
-                keccak256(bytecode)
+                keccak256(abi.encodePacked(
+                    type(ERC20ProxyDelegator).creationCode, 
+                    abi.encode(_token, _delegate)
+                ))
             )
-        );
-        return address(uint160(uint256(hash)));
+        ))));
     }
 }
```

---

### [G-05] Do not compute condition in every loop.

<a name="G-05"></a>
[To the top](#TOP)

There is not need to recalculate the max length in every iteration it is constant during execution function flow.

Deployment: **−1268**

Average Method Call: **214**

|      Contract      |    Method     |  Min  |   Max   |    Avg     | # calls | usd (avg) |
| :----------------: | :-----------: | :---: | :-----: | :--------: | :-----: | :-------: |
| ERC20MultiDelegate | delegateMulti | 90037 | 879930  |   520272   |   19    |   4.10    |
| ERC20MultiDelegate |    setUri     |   -   |    -    |   32694    |    1    |   0.26    |
|    Deployments     |               |       |         | % of limit |         |           |
| ERC20MultiDelegate |       -       |   -   | 4123185 |   13.7 %   |  32.46  |           |

#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..5c0f7d6 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -81,10 +81,12 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

+        uint256 length = Math.max(sourcesLength, targetsLength);
+
        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
-            transferIndex < Math.max(sourcesLength, targetsLength);
+            transferIndex < length;
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength

```