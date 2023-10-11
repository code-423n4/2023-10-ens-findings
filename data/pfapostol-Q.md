## Summary table

<a name="TOP"></a>

The findings report provides insights and recommendations for improving the current implementation. It highlights areas for potential enhancement and suggests adjustments to optimize code efficiency, improve user experience, and ensure compliance with best practices. 

The report emphasizes the importance of streamlining code by removing redundancy, providing clear documentation and error messages, and optimizing the usage of type casting. 

Taking these recommendations into consideration will enhance the overall security, performance, and maintainability of the system. By addressing the findings, the implementation will be better equipped to meet user expectations, minimize potential errors, and provide a more robust and user-friendly experience.

|   Index    | Issue                                                                                  | Instances |
| :--------: | :------------------------------------------------------------------------------------- | :-------: |
| [1](#N-01) | [Prevent users from transfering zero tokens](#N-01)                                     |     3     |
| [2](#N-02) | [Redundant function](#N-02)                                                             |     1     |
| [3](#N-03) | [Avoid Using `this` to Call Own Functions](#N-03)                                       |     1     |
| [4](#N-04) | [Caller Must Implement ERC1155Receiver if Not an Externally Owned Account (EOA)](#N-04) |     4     |
| [5](#N-05) | [Insufficient Allowance for ERC20 Token Transfers](#N-05)                               |     1     |
| [6](#N-06) | [Delegation Failure Due to Insufficient Balance](#N-06)                                 |     1     |

11 instances over 6 issues

---

### [N-01] Disallow Transfer of Zero Tokens

<a name="N-01"></a>
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

### [N-02] Redundant function

<a name="N-02"></a>
[To the top](#TOP)

The delegateMulti function currently serves as a redundant intermediate step, as it simply calls the _delegateMulti function without any additional logic. To optimize performance and eliminate unnecessary jumps, it is recommended to merge these two functions into a single streamlined implementation. This consolidation will lead to more efficient code execution.

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

### [N-03] Avoid Using `this` to Call Own Functions

<a name="N-03"></a>
[To the top](#TOP)

There is no need to explicitly cast ERC1155(this) in order to call your own functions. When executing functions within the same contract, you can directly call them without requiring the cast. Simply refer to the function by its name and pass the required arguments, as per the function's definition. This approach eliminates the need for unnecessary type casting and simplifies your code.

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 195](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L195)

<details>

<summary>see 1 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
Line 195: 195        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```

</details>

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..1a6dd6c 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -192,7 +192,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
     function getBalanceForDelegate(
         address delegate
     ) internal view returns (uint256) {
-        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+        return balanceOf(msg.sender, uint256(uint160(delegate)));
     }

     function retrieveProxyContractAddress(
```

---

### [N-04] Caller Must Implement ERC1155Receiver if Not an Externally Owned Account (EOA)

<a name="N-04"></a>
[To the top](#TOP)

In the _mintBatch function, if the caller of the contract is a contract (not an externally owned account), it is required that the caller implements the IERC1155Receiver interface. This check ensures that the caller can properly receive the tokens being minted.

To avoid confusion among developers integrating this contract, it is advisable to add developer comments either to the ERC20MultiDelegate contract or the delegateMulti function. These comments should clarify the requirement of implementing the IERC1155Receiver interface in case the caller is a contract. By providing clear documentation, developers will have a better understanding of the contract's behavior and requirements, minimizing potential confusion or misunderstandings.

---

### [N-05] Insufficient Allowance for ERC20 Token Transfers

<a name="N-05"></a>
[To the top](#TOP)

In the context of `ERC20` tokens, the `approve()` function is used to authorize another account to transfer a specific amount of tokens on behalf of the token holder. This is an essential step in the `ERC20` standard, particularly for secure delegated transfers in decentralized exchanges or DeFi protocols.

After being approved, the authorized account can transfer tokens up to the approved amount by using the `transferFrom()` function. However, if `transferFrom()` is called before the token holder has granted approval, the transaction will fail since the `ERC20` contract verifies the caller's allowance.

To ensure the secure movement of ERC20 tokens on behalf of another account, it is highly recommended to check if the allowance is sufficient before attempting the transfer. If the allowance is not adequate, you should throw a user-friendly error or revert the transaction, providing clear feedback to the user. This helps prevent unauthorized or erroneous transfers and maintains the security and functionality of ERC20 tokens.

It's important to prioritize the safety and security of token holders' funds, and by implementing proper allowance checks, you can enhance the robustness of your ERC20 token contract

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 155-161](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161)

<details>

<summary>see 1 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
155	    function createProxyDelegatorAndTransfer(
156	        address target,
157	        uint256 amount
158	    ) internal {
159	        address proxyAddress = deployProxyDelegatorIfNeeded(target);
160	        token.transferFrom(msg.sender, proxyAddress, amount);
161	    }

```

</details>

---

### [N-06] Delegation Failure Due to Insufficient Balance

<a name="N-06"></a>
[To the top](#TOP)

`createProxyDelegatorAndTransfer` will revert if there is not enough balance, check in advance and revert earlier with meaningfull error

#### <ins>Proof Of Concept</ins>

[Source: ERC20MultiDelegate.sol, lines 155-161](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161)

<details>

<summary>see 1 instances</summary>

#### File project/2023-10-ens/contracts/ERC20MultiDelegate.sol: 

```solidity
155	    function createProxyDelegatorAndTransfer(
156	        address target,
157	        uint256 amount
158	    ) internal {
159	        address proxyAddress = deployProxyDelegatorIfNeeded(target);
160	        token.transferFrom(msg.sender, proxyAddress, amount);
161	    }

```

</details>

#### Fix:

```diff
diff --git a/contracts/ERC20MultiDelegate.sol b/contracts/ERC20MultiDelegate.sol
index 28e0f5d..39a1738 100644
--- a/contracts/ERC20MultiDelegate.sol
+++ b/contracts/ERC20MultiDelegate.sol
@@ -35,6 +35,8 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         address indexed to,
         uint256 amount
     );
+
+    error BalanceNotEnough();

     /**
      * @dev Constructor.
@@ -157,6 +159,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         uint256 amount
     ) internal {
         address proxyAddress = deployProxyDelegatorIfNeeded(target);
+        if (token.balanceOf(msg.sender) < amount) revert BalanceNotEnough();
         token.transferFrom(msg.sender, proxyAddress, amount);
     }
```