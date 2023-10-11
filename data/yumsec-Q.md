# QA Report

## Q-01: Function _createProxyDelegatorAndTransfer doesn’t assert sufficient balance

### Lines of Code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155-L161

### Proof of Concept
unlike function `_processDelegation`, function `createProxyDelegatorAndTransfer` doesn't check balance before token transfer.

```solidity
function createProxyDelegatorAndTransfer(
    address target,
    uint256 amount
) internal {
    address proxyAddress = deployProxyDelegatorIfNeeded(target);
    token.transferFrom(msg.sender, proxyAddress, amount);
}
```

### Mitigation

Check balance of msg.sender before transferring token.

```solidity
function createProxyDelegatorAndTransfer(
    address target,
    uint256 amount
) internal {
    assert(amount <= balanceOf(msg.sender));
    address proxyAddress = deployProxyDelegatorIfNeeded(target);
    token.transferFrom(msg.sender, proxyAddress, amount);
}
```

## Q-02: Function _reimburse doesn’t assert sufficient balance

### Lines of Code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144-L147

### Proof of Concept
unlike function `_processDelegation`, function `_reimburse` doesn't check token balance of `source` before transferring token.

```solidity
function _reimburse(address source, uint256 amount) internal {
    // Transfer the remaining source amount or the full source amount
    // (if no remaining amount) to the delegator
    address proxyAddressFrom = retrieveProxyContractAddress(token, source);
    token.transferFrom(proxyAddressFrom, msg.sender, amount);
}
```

### Mitigation

Check source balance before transferring token.

```solidity
function _reimburse(address source, uint256 amount) internal {
    uint256 balance = getBalanceForDelegate(source);
    assert(amount <= balance);

    // Transfer the remaining source amount or the full source amount
    // (if no remaining amount) to the delegator
    address proxyAddressFrom = retrieveProxyContractAddress(token, source);
    token.transferFrom(proxyAddressFrom, msg.sender, amount);
}
```