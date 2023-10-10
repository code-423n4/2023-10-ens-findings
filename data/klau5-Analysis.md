# Analysis of the code base

The existing contract ENSToken, inherits OpenZeppelin's ERC20Votes contract. With the OpenZeppelin ERC20Votes, user can delegate to only one address the voting power for the entire ERC20 balance of user. This project allows user to partially delegate voting power, and delegate to multiple addresses.

User can delegate voting power by depositing ERC20 into an ERC20ProxyDelegator contract. 

The delegated voting stake is represented by an ERC1155, which uses the delegate's address converted to a number as an ID. If user want to revoke delegation, user can burn the ERC1155 and get your ERC20 back.

## ERC20ProxyDelegator

It is deployed by ERC20MultiDelegate contract. The salt is fixed to 0, and the delegatee to which the ERC20 votes deposited in the contract are to be delegated is set by constructor parameter. Only one ERC20ProxyDelegator can be created for each delegatee.

```solidity
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}
```

## ERC20MultiDelegate

Send ERC20 to the ERC20ProxyDelegator and delegate voting rights. Shares for voting rights are minted in the form of ERC1155.

A user who wants to deposit their own ERC20 and delegate their voting rights calls the `delegateMulti` function. The length of `sources` and `targets` can be different, but the length of `amounts` must be equal to the longer of the two.

If the `sources` and `targets` match, move the ERC20 between the ERC20ProxyDelegator contracts. Burn ERC1155 of source, and mint ERC1155 of target. If `sources` is longer, transfer the ERC20 deposited at ERC20ProxyDelegator of source to msg.sender and burn msg.sender's ERC1155. If `targets` is longer, deploy ERC20ProxyDelegator of target, deposit ERC20, and mint ERC1155 to msg.sender.

```solidity
function delegateMulti(
  uint256[] calldata sources,
  uint256[] calldata targets,
  uint256[] calldata amounts
) external {
    _delegateMulti(sources, targets, amounts);
}

function _delegateMulti(
  uint256[] calldata sources,
  uint256[] calldata targets,
  uint256[] calldata amounts
) internal {
  uint256 sourcesLength = sources.length;
  uint256 targetsLength = targets.length;
  uint256 amountsLength = amounts.length;

  require(
      sourcesLength > 0 || targetsLength > 0,
      "Delegate: You should provide at least one source or one target delegate"
  );

  require(
      Math.max(sourcesLength, targetsLength) == amountsLength,
      "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
  );
...
}
```

Minting or burning ERC1155 is handled in batches after other tasks in the `_delegateMulti` function have finished.

```solidity
function _delegateMulti(
  uint256[] calldata sources,
  uint256[] calldata targets,
  uint256[] calldata amounts
) internal {
  ...
  if (sourcesLength > 0) {
      _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
  }
  if (targetsLength > 0) {
      _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
  }
}
```

### _processDelegation

If `sources` and `targets` match, move delegated vote power from source to target by transfering ERC20 from source's ERC20ProxyDelegator to target's ERC20ProxyDelegator.

```solidity
function _delegateMulti(
  uint256[] calldata sources,
  uint256[] calldata targets,
  uint256[] calldata amounts
) internal {
  ...

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
      }
      ...
  }
  ...
}
```

Transfer ERC20 between two ERC20ProxyDelegator contracts in `_processDelegation`. To move the ERC20 deposited in source, msg.sender must own the sufficient amount of ERC1155 minted with the `source` ID. This ERC1155 will be burned later.

In the `transferBetweenDelegators` function, `transferFrom` is used to trasnfer ERC20. ERC20MultiDelegate is approved when ERC20ProxyDelegator deployed.

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

function getBalanceForDelegate(
    address delegate
) internal view returns (uint256) {
    return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
}

function transferBetweenDelegators(
    address from,
    address to,
    uint256 amount
) internal {
    address proxyAddressFrom = retrieveProxyContractAddress(token, from);
    address proxyAddressTo = retrieveProxyContractAddress(token, to);
    token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
}
```

If the receiver's ERC20ProxyDelegator is not already deployed, it deploy new contract. The salt is fixed at 0 but is deployed to different addresses by the delegate addresses passed to the constructor. Compute the contract address and deploy the contract if it is not already deployed at that address.

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

### _reimburse

If the `sources` array is longer than `targets`, it will burn the ERC1155 owned by msg.sender for `sources` that longer than `targets` and get back the ERC20, and revoke the delegation.

```solidity
function _delegateMulti(
  uint256[] calldata sources,
  uint256[] calldata targets,
  uint256[] calldata amounts
) internal {
  ...

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

    ...
    else if (transferIndex < sourcesLength) {
        // Handle any remaining source amounts after the transfer process.
        _reimburse(source, amount);
  }
  ...
}
```

Unlike the original ERC20Vote, this project can revoke partial of delegation. ERC1155 burns are processed all at once at the end, after all tasks have been completed.

```solidity
function _reimburse(address source, uint256 amount) internal {
  // Transfer the remaining source amount or the full source amount
  // (if no remaining amount) to the delegator
  address proxyAddressFrom = retrieveProxyContractAddress(token, source);
  token.transferFrom(proxyAddressFrom, msg.sender, amount);
}
```

### createProxyDelegatorAndTransfer

If the length of `targets` is longer than `sources`, deploy an ERC20ProxyDelegator that delegates votes to the remaining `targets` and deposits ERC20 into it.

```solidity
function _delegateMulti(
    uint256[] calldata sources,
    uint256[] calldata targets,
    uint256[] calldata amounts
  ) internal {
    ...

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

        ...
        else if (transferIndex < targetsLength) {
            // Handle any remaining target amounts after the transfer process.
            createProxyDelegatorAndTransfer(target, amount);
        }
    }
    ...
}
```

You will receive ERC1155 as much as you deposit. All ERC1155 mint operations are processed together at the end of the `_delegateMulti`.

```solidity
function createProxyDelegatorAndTransfer(
    address target,
    uint256 amount
) internal {
    address proxyAddress = deployProxyDelegatorIfNeeded(target);
    token.transferFrom(msg.sender, proxyAddress, amount);
}
```

### Time spent:
9 hours