
# QA Report

## Low Risk Issues
| Count | Explanation | 
|:--:|:-------|
| [L-01] | Using hardcoded `salt` value can be problematic |
| [L-02] | Unsafe ERC20 transfer methods |

| Total Low Risk Issues | 2 |
|:--:|:--:|

## Non-Critical Issues
| Count | Explanation | 
|:--:|:-------|
| [N-01] | Unnecessary checks in `_processDelegation` |  

| Total Non-Critical Issues | 1 |
|:--:|:--:|

### [L-01] Using hardcoded `salt` value can be problematic

In the code, the `salt` value is hard-coded to `uint256(0)`, which means that it's always using the same `salt` value for every deployment. This is not a recommended practice for predictable contract deployments because it can lead to address collisions and is against best practices.

```solidity
File: ERC20MultiDelegate.sol

    if (bytecodeSize == 0) {
@->     new ERC20ProxyDelegator{salt: 0}(token, target);
        emit ProxyDeployed(target, proxyAddress);
    }

```

```solidity
File: ERC20MultiDelegate.sol

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
@->             uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }

```

In a typical usage of predictable address generation, one should ensure that the `salt` value is unique for each contract deployment to avoid collisions. The `salt` value is often incremented for each deployment, ensuring that the resulting contract address is different each time.

#### Recommendation

Pass a unique `salt` value for each deployment, ensuring that the resulting contract addresses are distinct.

### [L-02] Unsafe ERC20 transfer methods

With `ENSToken`, there is no need to use safe transfer but There are many [Weird ERC20](https://github.com/d-xo/weird-erc20) Tokens that won't work correctly.

In case, the contract is used with wrapped token of Some tokens that do not revert on failure, but instead return false (e.g. ZRX, EURS), then even the failed transfer will look successful and create issues.

Consider using openzeppelin's `SafeERC20` for `transferFrom` in the contract.

### [N-01] Unnecessary checks in `_processDelegation`

In `_processDelegation`, there is a check to make sure that source has enough balance to transfer the delegation to target.

```solidity

  function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
@->     uint256 balance = getBalanceForDelegate(source);

@->     assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }

```

But it is unnecessary as if source doesn't have enough balance to transfer it will revert in the `_delegateMulti` as this line:

```solidity

  if (sourcesLength > 0) {
      _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
  }

```
