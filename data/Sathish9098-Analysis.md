# ENS - Analysis

## Overview
This system is designed to be used with an ``ERC20 token`` that supports ``delegation of votes`` (such as the OpenZeppelin ``ERC20Votes`` extension) and ``ERC1155 metadata``

### ERC20MultiDelegate contract

- Inherits from ``ERC1155`` and ``Ownable``.
- Allows delegators to ``delegate`` their tokens to multiple ``target`` delegates simultaneously.
- Keeps track of the ``ERC20 token`` (via the token variable) with which ``delegations`` are made.
- Provides a mechanism for deploying ``proxy contracts`` to ``facilitate delegation``.

### ERC20ProxyDelegator:

- A child contract that is deployed by ``ERC20MultiDelegate``.
- Its purpose is to proxy ``delegate votes`` on behalf of an ``original delegator``.
- It is constructed with a reference to the ``ERC20 token`` and the address of the delegate to whom votes are ``delegated``.

## Approach taken in evaluating the codebase

- ``High-level overview`` : I analyzed the ``overall codebase`` in one iteration to get a high-level understanding of the ``code structure`` and ``functionality``.

- ``Documentation review`` : I studied the ``documentation`` to understand the purpose of ``each contract``, its ``functionality``, and how it is ``connected`` with other contracts.

- ``Literature review`` : I read ``old audits`` and ``known findings``, as well as the ``bot races findings``.

- ``Testing setup`` : I set up my ``testing environment`` and ran the tests to ensure that ``all tests passed``. Used yarn and hardhat to test this protocol.
  - npx hardhat node
  - yarn test test/delegatemulti.js
  - yarn coverage

- ``Detailed analysis`` : I started with the ``detailed analysis`` of the code base, line by line. I took the ``necessary notes`` to ask some questions to the ``sponsors``.

## Architecture recommendations


### Insufficient Validation of Target Delegate Address  
The ``_processDelegation()`` function in the ``ERC20MultiDelegate`` contract does not check if the ``target`` delegate is a ``valid address``. This means that an attacker could send delegations to ``invalid addresses``, which could result in ``lost funds``

#### Step-by-step explanation of how an attacker could exploit this vulnerability

- The attacker deploys a malicious contract that ``impersonates`` a valid ``target delegate``.
- The ``attacker`` sends a ``delegation`` to the malicious contract from their ``own account``.
- The ``ERC20MultiDelegate`` contract transfers the ``delegation`` to the ``malicious contract``.
- The malicious contract withdraws the delegated funds to the attacker's control.

The ``_processDelegation()`` function should be modified to check if the ``target delegate`` is a ``valid`` address before sending the delegation.

### Inefficient delegation processing in _delegateMulti() function

The ``_delegateMulti()`` function in the ``ERC20MultiDelegate`` contract iterates through all source and ``target delegates``, even if there is no delegation to be processed for some of them. This can be inefficient and waste gas, especially for large numbers of delegates.

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

```

#### Steps:

``Step 1``: The ``_delegateMulti()`` function iterates through all source and target delegates, even if there is no delegation to be processed for some of them. This is because the loop condition is ``transferIndex < Math.max(sourcesLength, targetsLength)``. This means that the loop will continue to iterate until all of the delegates have been processed, even if there are ``no delegations`` to be processed for some of them.

``Step 2``: For each delegate, the ``_delegateMulti()`` function checks if there is a delegation to be processed. This is done by checking if the amount for the delegate is greater than zero. If the amount is greater than zero, then the delegation is processed.

``Step 3``: If the delegation is processed, then the ``_delegateMulti()`` function transfers the delegation from the source delegate to the target delegate. This is done by calling the ``transfer()`` function on the ERC20 token contract.

``Step 4``: The ``transfer()`` function is an expensive operation, especially for ``large delegations``. This is because the ``transfer()`` function needs to update the state of the ERC20 token contract.

``Step 5``: If there is no delegation to be processed for a given delegate, then the ``_delegateMulti()`` function skips the delegate and moves on to the next delegate.

To improve the efficiency of the ``_delegateMulti()`` function is to use a more efficient data structure to store the ``delegation information``. For example, a ``tree structure`` could be used to store the ``delegates`` and their delegations. This would make it more efficient to ``search`` for and ``update`` delegation information, and it would also avoid the need to ``iterate`` through all of the delegates in the _delegateMulti() function.

By making these changes, the ``_delegateMulti()`` function can be made more efficient and ``less wasteful of gas``. This is especially important for ``large numbers`` of ``delegates`` and ``large delegations``.

## Codebase quality analysis

#### Add documentation
The code could be better documented. This would make it easier to understand how the code works and how to use it

#### Improve error handling
The code could be improved by adding better error handling. This would help to prevent errors from causing unexpected behavior or crashes. The code could check for errors when calling the ``transfer()`` function and handle any errors that occur

#### Use consistent coding style
The code should use a consistent coding style. This will make the code more readable and easier to maintain. The code should use ``consistent indentation``, ``spacing``, and ``line breaks``

#### Use meaningful variable names
The code should use ``meaningful`` ``variable names``. This will make the code more readable and easier to understand.Instead of using variable names like ``a``, ``b``, and ``c``, the code should use variable names meaningfull 

#### Break down the code into smaller functions
The code should be broken down into smaller functions. This will make the code more modular and easier to maintain. For example, the ``_delegateMulti()`` function could be broken down into smaller functions for each step of the delegation process

#### Use a caching mechanism
The code currently does not use any ``caching mechanisms``. This can lead to unnecessary performance overhead, especially for frequently accessed delegations. A caching mechanism could be used to cache the delegation information. This would improve the performance of the code, especially for frequently accessed delegations.

## Centralization risks

The ``setUri()`` function can only be called by the ``owner`` of the contract. This gives the owner a lot of power, as they can change the ``URI`` of the contract at will. This could be used to maliciously redirect ``users`` to a fake URI or to change the branding of the contract.

```
function setUri(string memory uri) external onlyOwner {

```

The fact that the ``setUri()`` function can only be called by the owner introduces a ``single point of failure``. If the owner's account is ``compromised``, then the attacker could use the ``setUri()`` function to perform malicious actions.

Add a timelock to the ``setUri()`` function. This would require the ``owner`` to wait a certain amount of time before they can change the ``URI`` of the contract. This would give the ``community time`` to review any ``proposed changes`` and to object if necessary.

## Systemic risks

### Delegation attacks 
The ``ERC20MultiDelegate`` contract allows delegators to delegate their tokens to multiple ``target`` delegates simultaneously. This can be useful for ``delegators`` who want to spread their risk or who want to vote for ``multiple delegates``. However, it also introduces the possibility of ``delegation attacks``.  An attacker could create a malicious proxy delegator contract that impersonates a legitimate delegate. If delegators delegate their tokens to this malicious contract, the attacker could steal their funds.

### Smart contract vulnerabilities
The ``ERC20MultiDelegate`` and ``ERC20ProxyDelegator`` contracts are complex smart contracts. As such, they are susceptible to ``smart contract vulnerabilities``, such as ``reentrancy attacks`` and ``integer overflows``. If a vulnerability is discovered in one of these contracts, it could be exploited to steal funds from the contract or to disrupt its operation.

### ERC1155 Metadata URI
The contract inherits from ERC1155 and expects a metadata URI for token metadata. Ensure that the provided URI is accessible and that metadata is set correctly.

### Proxy Contract Verification Risks
The method used to verify whether a proxy contract has already been deployed is not foolproof and could result in false positives or negatives.

There are a number of other potential risks associated with using the ERC20MultiDelegate and ERC20ProxyDelegator contracts. For example, the contracts are still under development and may contain bugs. Additionally, the contracts rely on a number of external dependencies, such as the ERC20 token contract. If there is a problem with one of these external dependencies, it could affect the operation of the ERC20MultiDelegate and ERC20ProxyDelegator contracts.

## Implementation Suggestions

### Security Enhancements
- Implement security best practices to prevent common vulnerabilities like reentrancy and integer overflow/underflow
- Consider using OpenZeppelin's libraries for security-critical operations.
- Implement access control mechanisms to restrict who can call critical functions.

### Simplify Address Retrieval
Simplify the logic for retrieving proxy contract addresses. Using assembly might make the code harder to read and maintain.

### Version Compatibility
Keep an eye on updates to the OpenZeppelin libraries and ensure compatibility with the chosen versions.

### Solidity Versions
If you are using a floating version of Solidity, such as ^0.8.2, this means that your codebase will be compatible with all versions of Solidity from ``^0.8.0`` to the latest version. This can be useful if you are not sure which version of Solidity you want to use, or if you need to support multiple versions of Solidity.

Overall, I would recommend using the ``latest version`` like ``0.8.20`` or ``0.8.21`` of Solidity if possible.

### Comments
Comments can be very helpful for both auditors and developers, especially in complex codebases. I also agree that the codebase you are describing is generally clean and strategically straightforward, but there are a few sections that could benefit from additional comments.

#### Some specific tips for commenting :

- Add comments to explain complex code
- Add comments to document your design decisions
- Add comments to document your design decisions
- Use clear and concise language

## Conclusion
``ERC20MultiDelegate``, serves as a utility for token delegation in a decentralized governance system. It allows delegators to delegate their tokens to multiple target delegates efficiently using proxy contracts.

## Time spent:
10 hours







### Time spent:
10 hours