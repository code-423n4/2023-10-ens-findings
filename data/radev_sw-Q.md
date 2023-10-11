# QA Report - ENS Audit Contest | 5 Oct 2023 - 11 Oct 2023

---

# Executive summary

### Overview

|              |                                           |
| :----------- | :---------------------------------------- |
| Project Name | ENS                                       |
| Repository   | https://github.com/code-423n4/2023-10-ens |
| Website      | [Link](https://ens.domains/)              |
| Twitter      | [Link](https://twitter.com/ensdomains)    |
| Methods      | Manual review                             |

### Scope

| Contract (1)                                                                                                             | SLOC | Purpose                                                                | Libraries used                                           |
| :----------------------------------------------------------------------------------------------------------------------- | :--- | :--------------------------------------------------------------------- | :------------------------------------------------------- |
| [contracts/ERC20MultiDelegate.sol](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) | 216  | ERC20Votes compatible multi-delegation contract to manage user votings | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

---

---

# Findings Summary

| ID              | Title                                                                                                                                      | Instances | Severity                  |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------- | ------------------------- |
| [L-01](#L-01)   | Inefficient token handling in `delegateMulti()` when a user wants to both `reimburse` and `delegate` MultiDelegate tokens                  | -         | _`Low to Medium`_         |
| [L-02](#L-02)   | `ERC20MultiDelegate` contract breaks ERC1155 specifications                                                                                | -         | _`Low to Medium`_         |
| [L-03](#L-03)   | Unbounded `for` loop in `_delegateMulti()`. Set max value for the loop iterations (max values for `sourcesLength[]` and `targetsLength[]`) | 1         | _Low_                     |
| [L-04](#L-04)   | Static Salt in `deployProxyDelegatorIfNeeded()`: Possibility of DoS due to Predictable Contract Address                                    | 1         | _Low_                     |
| [L-05](#L-05)   | Unchecked Initialization in `deployProxyDelegatorIfNeeded()`: Risk of DoS due to Potential Malfunction in `ERC20ProxyDelegator`            | 1         | _Low_                     |
| [L-06](#L-06)   | Front Running of `ERC20ProxyDelegator` Deployment                                                                                          | 1         | _Low_                     |
| [L-07](#L-07)   | Gas Concerns - DoS in `deployProxyDelegatorIfNeeded()`                                                                                     | 1         | _Low_                     |
| [NC-01](#NC-01) | Potential Reversion Issue with Certain ERC20 Token Approvals                                                                               | 1         | _Non Critical_            |
| [NC-02](#NC-02) | Unchecked Return Values for `approve()`                                                                                                    | 1         | _Non Critical_            |
| [NC-03](#NC-03) | Need for Comments on State Variables                                                                                                       | 1         | _Non Critical_            |
| [NC-04](#NC-04) | Absence of Event Emissions                                                                                                                 | 3         | _Non Critical_            |
| [NC-05](#NC-05) | Missing address zero checks for `sources[]` and `targets[]` arrays in `delegateMulti()` function                                           | 1         | _Non Critical_            |
| [S-01](#S-01)   | Optimization for `deployProxyDelegatorIfNeeded()` function logic                                                                           | -         | _Suggestion/Optimization_ |
| [S-02](#S-02)   | Optimization for transferring flow                                                                                                         | -         | _Suggestion/Optimization_ |

**Note: I will leave it to the judge and sponsor to determine the actual severity of issues that I have classified as `Severity: Low to Medium`.**

---

---

## <a name="L-01"></a>[L-01] Inefficient token handling in `delegateMulti()` when a user wants to both `reimburse` and `delegate` MultiDelegate tokens

### GitHub Links:

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57-L116
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L124-L137
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155-L161

### Impact

- Inefficient token handling in `delegateMulti()` when a user wants to both `reimburse` and `delegate` MultiDelegate tokens / Inefficient System Design and Flow.
- Users left with a negative impression of the protocol.
- Users spend more gas.
- The user who is delegator ends up with inefficient minted MultiDelegate tokens, resulting in unfair `MultiDelegate` tokens delegation in the future.

### Proof of Concept

#### Scenario

1. Bob approve `ERC20MultiDelegate` contract with `100000 ERC20Votes tokens` and after that `delegate 100000 ERC20Votes to Alice`. For this purpose he calls `delegateMulti()` function with the following parameters.

   - `sources[]`: []
   - `targets[]`: [alice's address]
   - `arguments[]`: [100000]

2. After some period of time, Bob decides to `reimburse` his 100,000 ERC20Votes tokens to Alice and `delegate` another 100,000 ERC20Votes tokens to Charlie in a `single transaction`.
   - For this purpose, Bob calls `delegateMulti(sources: [alice's address], targets: [bob's address, charlie's address], amounts: [100000, 100000])`.
   - As a result, Bob removes his delegated tokens to Alice and delegates new 100,000 ERC20Votes to Charlie. However...
   - However Bob doesn't actually receive back his 100,000 ERC20Votes tokens. Instead, these tokens are transferred to the ERC20ProxyDelegate contract.
   - Consequently, Bob is left with a negative impression of the protocol and has to execute another transaction (spend additional gas) to reclaim his ERC20Votes tokens.

#### Coded Proof of Concept

- Paste the following test in _2023-10-ens\test\delegatemulti.js_, _describe("deposit")_
- **Also paste this code block below after this line: https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L130**

```javascript
await increaseTime(365 * 24 * 60 * 60);
await token.mint(bob, mintAmount);
await increaseTime(365 * 24 * 60 * 60);
await token.mint(alice, mintAmount);
```

---

```javascript
it("proof of concept", async () => {
  // For our example let's say that:
  // Bob: delegator

  const bobSigner = await ethers.provider.getSigner(bob);

  // -> Step 1: Bob approve ERC20MultiDelegate contract with 100000 ERC20Votes tokens and after that delegate 100000 ERC20Votes to Alice
  await token.connect(bobSigner).approve(multiDelegate.address, 100000);
  await multiDelegate.connect(bobSigner).delegateMulti([], [alice], [100000]);

  let bobToAliceMultiDelegateTokensBalance = await multiDelegate.balanceOf(
    bob,
    alice
  );
  console.log(
    `Step 1. MultiDelegated tokens from Bob to Alice: ${bobToAliceMultiDelegateTokensBalance.toString()}`
  ); // 100000
  expect(await bobToAliceMultiDelegateTokensBalance.toString()).to.equal(
    "100000"
  );

  // -> Step 2: Bob decides to `reimburse` his 100,000 ERC20Votes tokens to Alice and `delegate` another 100,000 ERC20Votes tokens to Charlie in a single transaction.
  // For this purpose, Bob calls `delegateMulti(sources: [alice], targets: [bob, charlie], amounts: [100000, 100000])`.
  // As a result, Bob removes his delegated tokens to Alice and delegates new 100,000 ERC20Votes to Charlie. However...
  // However Bob doesn't actually receive back his 100,000 ERC20Votes tokens. Instead, these tokens are transferred to the ERC20ProxyDelegate contract.
  // Consequently, Bob is left with a negative impression of the protocol and has to execute another transaction (spend additional gas) to reclaim his ERC20Votes tokens.
  await token.connect(bobSigner).approve(multiDelegate.address, 100000);
  await multiDelegate
    .connect(bobSigner)
    .delegateMulti([alice], [bob, charlie], [100000, 100000]);

  bobToAliceMultiDelegateTokensBalance = await multiDelegate.balanceOf(
    bob,
    alice
  );
  let bobToCharlieMultiDelegateTokensBalance = await multiDelegate.balanceOf(
    bob,
    charlie
  );
  let bobToBobMultiDelegateTokensBalance = await multiDelegate.balanceOf(
    bob,
    bob
  );
  console.log(
    `Step 2. MultiDelegated tokens from Bob to Alice: ${bobToAliceMultiDelegateTokensBalance.toString()}`
  ); // 0
  console.log(
    `Step 2. MultiDelegated tokens from Bob to Charlie: ${bobToCharlieMultiDelegateTokensBalance.toString()}`
  ); // 100000
  console.log(
    `Step 2. MultiDelegated tokens from Bob to Bob: ${bobToBobMultiDelegateTokensBalance.toString()}`
  ); // 100000

  expect(await bobToAliceMultiDelegateTokensBalance.toString()).to.equal("0");
  expect(await bobToCharlieMultiDelegateTokensBalance.toString()).to.equal(
    "100000"
  );
  expect(await bobToBobMultiDelegateTokensBalance.toString()).to.equal(
    "100000"
  );
});
```

### Tools Used

- Manual inspection
- Hardhat

---

## <a name="L-02"></a>[L-02] `ERC20MultiDelegate` contract breaks ERC1155 specifications

### GitHub Links

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L25-L216

### Impact

- `ERC20MultiDelegate` contract does not adhere to ERC1155.
- Smart Contract Interaction Breakdown: Any smart contract expecting standard ERC1155 behavior might not function correctly when interacting with the `ERC20MultiDelegate.sol` contract.
- If tokens can be minted, burned or transferred to contracts without ensuring that they can accept ERC1155 tokens, it might expose the token and its holders to certain risks.

### Proof of Concept

Description: The `ERC20MultiDelegate` contract implements ERC1155 (https://eips.ethereum.org/EIPS/eip-1155). This contract contains `delegateMulti()` function that facilitates the process of transferring delegation amounts across multiple source and target delegate pairs. It serves as an interface for the `_delegateMulti()` function, which contains the actual logic.

Inside the `_delegateMulti()` function MultiDelegate tokens are minted and burned (transferred) via `_mintBatch()` and `_burnBatch()` functions.

Whenever there is a transfer, the standard requires checking the receiver accepts the transfer:

- "If an implementation specific API function is used to transfer ERC-1155 token(s) to a contract, the safeTransferFrom or safeBatchTransferFrom (as appropriate) rules MUST still be followed if the receiver implements the ERC1155TokenReceiver interface. If it does not the non-standard implementation SHOULD revert but MAY proceed." By not checking a contract receiver accepts the transfer, `ERC20MultiDelegate` contract does not adhere to ERC1155.

### Tools Used

- Manual Inspection

### Recommended Mitigation Steps

If the recipient implements ERC1155TokenReceiver, require that it accepts the transfer. If the recipient is a contract that does not implement a receiver, reject the operation.

---

## <a name="L-03"></a>[L-03] Unbounded `for` loop in `_delegateMulti()`: Set maximum limit for loop iterations (max values for `sourcesLength` and `targetsLength`)

#### GitHub Links

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116
- (more specific link: https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85-L108)

#### Overview:

The function `_delegateMulti()` processes the delegation transfer process for multiple source and target delegates. Currently, there are no explicit bounds set for the lengths of the `sources` and `targets` arrays. This lack of upper limit poses potential risks in terms of gas costs and unwanted system behaviors.

#### Impact:

1. **Denial of Service DoS**

2. **Gas Costs:** If an excessively large number of delegates is supplied to the function, the unbounded loop can consume a lot of gas. In extreme cases, this might cause the transaction to fail due to reaching the block gas limit.

3. **Denial-of-Service (DoS) Attack Vector:** Malicious actors can attempt to exhaust the contract's functions by calling `_delegateMulti()` with a large number of delegates, potentially causing regular users' transactions to fail or become more expensive.

4. **Operational Delays:** For a contract that is intended to process other transactions in a timely manner, long-loop iterations can cause operational delays.

#### Recommendation:

To mitigate these risks, consider adding an explicit maximum bound for the lengths of the `sources[]` and `targets[]` arrays. This will ensure that the loop does not iterate an excessive number of times:

```solidity
uint256 constant MAX_DELEGATES = 100; // Example value; adjust as needed

require(sourcesLength <= MAX_DELEGATES, "Delegate: Too many source delegates provided");
require(targetsLength <= MAX_DELEGATES, "Delegate: Too many target delegates provided");
```

By introducing these checks, you can ensure that the `_delegateMulti()` function remains efficient and less susceptible to potential abuse.

---

## <a name="L-04"></a>[L-04] Static Salt in `deployProxyDelegatorIfNeeded()`: Possibility of DoS due to Predictable Contract Address

#### GitHub Links

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190

#### **Impact**

This vulnerability potentially impacts the upgradability of the `ERC20ProxyDelegator` contract. In scenarios where there's a need to redeploy an `ERC20ProxyDelegator` for a specific `token` and `delegate` combination due to, for instance, an upgrade or bug fix, the system wouldn't be able to deploy the new contract at the same address. As a result, the predictability and management of the system can be compromised. Furthermore, an attacker aware of this static salt can precompute and self-destruct the proxy contract's address, causing a Denial-of-Service (DoS) condition where the contract can't be redeployed at its expected address.

#### **Explanation**

The `deployProxyDelegatorIfNeeded` function is responsible for deploying a new `ERC20ProxyDelegator` contract, if not already deployed, for a specific `token` and `delegate` combination. The deployment uses the `CREATE2` opcode with a static salt (`uint256(0)`). As a result, the address for a new proxy contract is predictable and always the same for the given `token` and `delegate` pair.

The `ERC20ProxyDelegator` contract is a proxy delegator contract designed to vote on behalf of the original delegator. Upon deployment, it automatically delegates the votes and approves the utility contract to spend its tokens.

#### **Proof of Concept**

1. Deploy an `ERC20ProxyDelegator` for a specific `token` and `delegate`.
2. Note the contract's address (it will be predictable due to the static salt).
3. Destroy or self-destruct the deployed `ERC20ProxyDelegator`.
4. Try to deploy the `ERC20ProxyDelegator` again for the same `token` and `delegate`. Since the contract's address is predictable and static, the system will try to deploy at the same address, which is already taken, leading to a failure in deployment.

#### **Recommendation Steps**

1. **Dynamic Salting**: Avoid using a static salt for the `CREATE2` opcode. Instead, employ a mechanism to dynamically generate the salt, for example, by using a combination of the current block number, timestamp, or a nonce which gets incremented with every deployment.
2. **Management Mechanism**: Implement a mechanism or function that allows the system admin or governance mechanism to update or modify the salt, ensuring that in scenarios where redeployment is needed, it can be managed efficiently.

---

## <a name="L-05"></a>[L-05] Unchecked Initialization in `deployProxyDelegatorIfNeeded()`: Risk of DoS due to Potential Malfunction in `ERC20ProxyDelegator`

#### GitHub Links

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190

#### **Impact**

If there's a malfunction during the initialization of the `ERC20ProxyDelegator`, or if an out-of-gas error occurs during its deployment, it could lead to the `ERC20ProxyDelegator` contract being deployed without proper initialization. This can render the deployed contract useless, potentially halting any subsequent operations or interactions relying on this contract. Such a scenario can result in a Denial-of-Service (DoS) state for the affected operations.

#### **Explanation**

The function `deployProxyDelegatorIfNeeded` is tasked with deploying a new `ERC20ProxyDelegator` contract when required. However, once the contract is deployed using the `new` keyword, the function doesn't verify if the newly deployed contract has been correctly initialized.

The `ERC20ProxyDelegator` contract, upon deployment, is expected to approve a specified amount of tokens for spending and delegate votes on behalf of the original delegator. If this initialization fails, the proxy delegator might not function as expected, causing subsequent operations to be disrupted.

#### **Proof of Concept**

1. Introduce an intentional failure in the `ERC20ProxyDelegator` constructor, such as an invalid operation or an out-of-gas consuming loop.
2. Try deploying the `ERC20ProxyDelegator` using the `deployProxyDelegatorIfNeeded` function.
3. Observe that even though the `ERC20ProxyDelegator` contract gets deployed, it might not be correctly initialized, making it non-functional or leaving it in an undesirable state.

#### **Recommendation Steps**

1. **Post-deployment Verification**: After deploying the `ERC20ProxyDelegator`, verify its state. For instance, check if the expected approvals and delegations have been made. If not, the deployment should be considered a failure, and corrective measures should be taken.

---

## <a name="L-06"></a>[L-06] Front Running of `ERC20ProxyDelegator` Deployment

#### GitHub Links

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L15-L20
- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190

#### Impact

An attacker can front-run the deployment of the `ERC20ProxyDelegator`, leading to unexpected behavior for the original transaction sender. The contract, instead of being deployed by the legitimate transaction, gets deployed by the attacker's transaction. This could lead to scenarios where the expected contract logic doesn't apply as intended, potentially causing monetary or functionality disruptions for users.

#### Explanation

Front-running is a scenario on public blockchains where a malicious actor can see pending transactions and then decide to place their own transaction with a higher gas price, ensuring it gets mined before the original transaction. In the context of the `ERC20ProxyDelegator` deployment:

- A malicious actor observes a transaction attempting to deploy an `ERC20ProxyDelegator` contract.
- Before the original transaction is mined, the attacker sends a similar transaction but with a higher gas price, aiming to have their transaction processed first.
- If successful, the `ERC20ProxyDelegator` is deployed by the attacker's transaction, not the original transaction. This can cause confusion and unexpected behavior, especially if the attacker's deployment has any deviations or manipulations.

#### Proof of Concept

1. Observe the mempool for pending transactions aiming to deploy `ERC20ProxyDelegator`.
2. When such a transaction is spotted, quickly send a similar transaction with a higher gas price.
3. If the network processes the attacker's transaction first, the `ERC20ProxyDelegator` contract will be deployed by the attacker's transaction, causing the original transaction to fail or behave unexpectedly.

#### Recommendation Steps

1. **Dynamic Salting**: Use dynamic salt values for deploying the proxy. By ensuring a unique salt for each deployment, the exact contract address becomes unpredictable, making front-running attempts harder.
2. **Commit-Reveal Scheme**: Implement a two-step deployment process. In the first step, users send a commitment (a hash of some random value). In the second step, they reveal the random value. Only after a successful reveal, the deployment is allowed. This ensures that front-runners can't determine the exact deployment details from the initial transaction.

---

## <a name="L-07"></a>[L-07] Gas Concerns - DoS in `deployProxyDelegatorIfNeeded()`

#### GitHub Links

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190

#### Impact

Excessive and unnecessary contract deployments could result in draining funds from the deploying address due to the high gas costs associated with contract creation. An attacker could intentionally cause frequent deployments, leading to increased transaction costs and potentially depleting the funds of the contract or the user that's bearing the deployment costs.

#### Explanation

Contract deployment on the Ethereum network consumes a considerable amount of gas, especially when the contract contains complex logic or storage. In the provided code, the `deployProxyDelegatorIfNeeded()` function deploys an instance of the `ERC20ProxyDelegator` contract. If there are pathways in the system that allow this function to be triggered without necessary conditions, it exposes a vulnerability.

A malicious actor might discover ways to frequently call these pathways, forcing the system to deploy new contract instances repeatedly. Each deployment incurs gas costs, and over time, this can be a vector for a Denial of Service (DoS) attack, as the deploying entity might run out of funds to cover these costs.

#### Proof of Concept

1. Identify pathways in the system where `deployProxyDelegatorIfNeeded()` can be triggered, preferably without stringent conditions or checks.
2. Write a script or manually send transactions that exploit these pathways, causing the system to deploy `ERC20ProxyDelegator` instances repeatedly.
3. Monitor the deploying address's balance. With each deployment, there should be a noticeable decrease in funds due to the gas fees.

#### Recommendation Steps

1. **Throttling**: Implement a mechanism to limit the frequency of deployments. For example, allow only one deployment per user within a set time window.

---

---

## <a name="NC-01"></a>[NC-01] Potential Reversion Issue with Certain ERC20 Token Approvals

Several well-known ERC20 tokens, such as `UNI` and `COMP`, may revert when the `approve` functions are called with values exceeding `uint96`. This behavior deviates from the standard ERC20 implementation and might lead to unexpected results.

For instance, both `UNI` and `COMP` tokens feature special-case logic in their `approve` functions. When the approval amount is `uint256(-1)`, the `allowance` is set to `type(uint96).max`. Systems that anticipate the approved value to match the value in the `allowances` mapping might face compatibility issues.

<i>This issue is present in the following location:</i>

```solidity
17:         _token.approve(msg.sender, type(uint256).max);
```

- Links: https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L17

**Full Contract Context**:

```solidity
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}
```

---

## <a name="NC-02"></a>[NC-02] Unchecked Return Values for `approve()`

It is essential to note that not all `IERC20` implementations utilize the `revert()` mechanism to indicate failures in the `approve()` function. Some instead return a boolean to signal any issues. By not verifying this returned value, certain operations might proceed as if they were successful, despite no actual approvals taking place.

<i>The issue occurs in the following segment:</i>

```solidity
17:         _token.approve(msg.sender, type(uint256).max);
```

Link: https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L17-L17

---

## <a name="NC-03"></a>[NC-03] Need for Comments on State Variables

To enhance readability and assist future reviewers, it is advisable to provide detailed comments on crucial state variables. Such documentation clarifies their intended roles and usage.

<i>This recommendation is based on the following code:</i>

```solidity
28:     ERC20Votes public token;
```

Link: https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol/#L28-L28

---

## <a name="NC-04"></a>[NC-04] Absence of Event Emissions

Event emissions play a vital role in tracking and auditing on-chain activities. The following functions currently lack event emissions, which may hinder transparency and monitoring:

- [`delegateMulti() / _delegateMulti()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116)
- [`reimburse()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144-L149)
- [`createProxyDelegatorAndTransfer()`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155-L161)

Consider introducing appropriate events to facilitate a more transparent operation flow.

---

#### Event for `delegateMulti()` / `_delegateMulti()`

For a function that is dealing with multiple delegations, it would be handy to have events that provide a clear indication of each delegation that took place:

```solidity
event Delegated(address indexed source, address indexed target, uint256 amount);
```

This event will be emitted within the `_processDelegation` function each time a delegation takes place. If `_processDelegation` is a function in the same contract, you would insert `emit Delegated(source, target, amount);` within that function.

---

#### Event for `_reimburse()`

This function essentially sends the un-delegated or remaining tokens back to the user. An event for this would look something like:

```solidity
event Reimbursed(address indexed source, address indexed user, uint256 amount);
```

You would emit this event within the `_reimburse` function right after the `transferFrom`:

```solidity
token.transferFrom(proxyAddressFrom, msg.sender, amount);
emit Reimbursed(source, msg.sender, amount);
```

---

#### Event for `createProxyDelegatorAndTransfer()`

This function seems to be creating a proxy and then transferring a specified amount of tokens to this proxy. The event for this would look like:

```solidity
event ProxyCreatedAndFunded(address indexed target, address proxyAddress, uint256 amount);
```

You would emit this event within the `createProxyDelegatorAndTransfer` function right after the `transferFrom`:

```solidity
token.transferFrom(msg.sender, proxyAddress, amount);
emit ProxyCreatedAndFunded(target, proxyAddress, amount);
```

---

## <a name="NC-05"></a>[NC-05] Missing address zero checks for `sources[]` and `targets[]` arrays in `delegateMulti()` function

#### Problem:

Within the `delegateMulti()` and `_delegateMulti()` functions, the `sources` and `targets` arrays contain Ethereum addresses. These addresses are further used in delegation operations. However, there is currently no validation to prevent the inclusion of the Ethereum zero address (`address(0)`) within these arrays. Using the zero address can lead to unexpected behaviors or errors, especially in delegation scenarios.

#### Code Snippet:

```solidity
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
    ...
}
```

[Link to Code](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116)

In the snippet above, there is logic to set `source` or `target` to `address(0)` if their respective lengths are exceeded. However, this does not prevent the user from manually providing the zero address in the `sources` or `targets` arrays.

#### Impact:

By not ensuring the absence of the Ethereum zero address in the `sources[]` and `targets[]` arrays, a number of potential risks arise:

1. **Irretrievable Token Loss:** If a user or developer mistakenly uses the zero address as a source or target, tokens may get sent to this address and become permanently inaccessible. This can lead to financial loss for users and a decrease in the total available supply of the token.
2. **Unexpected Behaviors:** Delegating to or from the zero address can lead to unpredictable results or system errors.

#### Suggested Resolution:

Introduce validation checks to ensure that none of the elements in the `sources[]` and `targets[]` arrays are equal to the Ethereum zero address. This can be done at the start of the `_delegateMulti()` function. The check can look something like:

```solidity
for (uint i = 0; i < sourcesLength; i++) {
    require(sources[i] != address(0), "Delegate: Source address cannot be the zero address");
}
for (uint j = 0; j < targetsLength; j++) {
    require(targets[j] != address(0), "Delegate: Target address cannot be the zero address");
}
```

By integrating this check, the contract can safeguard against accidental or malicious use of the Ethereum zero address within the delegation process.

---

---

## <a name="S-01"></a>[S-01] Optimization for `deployProxyDelegatorIfNeeded()` function logic

#### Optimization:

If the goal is only ever to retrieve the address (and not necessarily deploy every time), separating the retrieval logic and the deployment logic might be more efficient.

#### Code Snippet:

- **[deployProxyDelegatorIfNeeded()](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173-L190) function:**

```solidity
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```

- **[retrieveProxyContractAddress()](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L198-L215) function:**

```solidity
    function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode,
            abi.encode(_token, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }
```

## <a name="S-02"></a>[S-02] Optimization for transferring flow

#### Overview:

During the delegation transfer process for multiple source and target delegates, it's essential to maintain an efficient process to save on gas costs and optimize the overall transaction process. It's been identified that the current transfer flow can be optimized by introducing two additional checks:

- Ensure that `source` and `target` delegates are different.
- Ensure that the amount to be transferred is greater than zero.

#### Benefits of Optimization:

1. **Gas Efficiency:** Avoiding unnecessary transfers can save on gas costs, as each unnecessary operation within the Ethereum Virtual Machine (EVM) incurs gas costs.

2. **Logical Integrity:** Transferring tokens between identical source and target addresses or transferring an amount of zero doesn't align with the logical intent of a transfer operation.

3. **Preventing Unexpected Behaviors:** Ensuring logical checks can prevent potential unwanted system behaviors, edge cases, or vulnerabilities.

#### Links:

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65-L116)

#### Recommendation:

Incorporate the following checks before calling the functions that execute the actual transferring:

The modified section within the `_delegateMulti()` function can look like:

```solidity
// ... (rest of the code)

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
        if (source != target && amount > 0) {
            // Process the delegation transfer between the current source and target delegate pair.
            _processDelegation(source, target, amount);
        }
    } else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        if (source != address(0) && amount > 0) {
            _reimburse(source, amount);
        }
    } else if (transferIndex < targetsLength) {
        // Handle any remaining target amounts after the transfer process.
        if (target != address(0) && amount > 0) {
            createProxyDelegatorAndTransfer(target, amount);
        }
    }
}

// ... (rest of the code)

```

By implementing these checks, the contract becomes more optimized, reduces unnecessary operations, and ensures a logical flow of token transfers.

---
