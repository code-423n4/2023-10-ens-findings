# Gas report


## Unnecessary balance check
The balance check in `_processDelegation` is redundant
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124-L137
```solidity
function _processDelegation(
	address source,
	address target,
	uint256 amount
) internal {
	uint256 balance = getBalanceForDelegate(source);

	assert(amount <= balance);

	deployProxyDelegatorIfNeeded(target);
	transferBetweenDelegators(source, target, amount);

	emit DelegationProcessed(source, target, amount);
}
```
Because it duplicates the checks in [`_burnBatch`](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L111)
Please consider removing it.

## Math.max(sourcesLength, targetsLength) can be cached
Math.max is called several times, including in for-loop
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L80
```solidity
Math.max(sourcesLength, targetsLength) == amountsLength,
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87
```solidity
transferIndex < Math.max(sourcesLength, targetsLength);
```
Please consider saving it to a variable.


## Unnecessary deployment of proxy on 0 amount
The proxy will deploy even if the amount provided is 0.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L124-L137
```solidity
function _processDelegation(
	address source,
	address target,
	uint256 amount
) internal {
	uint256 balance = getBalanceForDelegate(source);

	assert(amount <= balance);

	deployProxyDelegatorIfNeeded(target);
	transferBetweenDelegators(source, target, amount);

	emit DelegationProcessed(source, target, amount);
}

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161
```solidity
function createProxyDelegatorAndTransfer(
	address target,
	uint256 amount
) internal {
	address proxyAddress = deployProxyDelegatorIfNeeded(target);
	token.transferFrom(msg.sender, proxyAddress, amount);
}
```
Please consider implementing a check for a 0 amount before initiating proxy deployment, if there's a possibility that users might input a 0 amount.
