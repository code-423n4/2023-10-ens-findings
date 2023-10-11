### Utilize [ERC-1167](https://eips.ethereum.org/EIPS/eip-1167) to save gas on deployment and address retrieval for each user
The current design deploys a new `ERC20ProxyDelegator` using CREATE2 for each user that receives ERC115 tokens or voting shares.
Moreover, the `ERC20ProxyDelegator` doesn't have any runtime code and is only deployed once. 
As contract creation is one of the most gas-expensive operations, this can be heavily optimized by using the [`ERC-1167` standard](https://eips.ethereum.org/EIPS/eip-1167).
As this requires several changes I will be providing a whole new implementation and a test file that proves the gas savings.

**Alternative Implementation**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/access/Ownable.sol";
import "@openzeppelin/token/ERC1155/ERC1155.sol";
import "@openzeppelin/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/utils/math/Math.sol";

import "@openzeppelin/proxy/Clones.sol";

contract ERC20ProxyDelegatorImplementation {
    ERC20Votes internal immutable token;
    address internal immutable owner;

    constructor(ERC20Votes _token) {
        token = _token;
        owner = msg.sender;
    }

    function initialize(address _delegate) external {
        require(msg.sender == owner);
        token.delegate(_delegate);
        token.approve(msg.sender, type(uint256).max);
    }
}

contract ERC20MultiDelegateProxyClones is ERC1155, Ownable {
    using Clones for address;

    ERC20Votes public token;
    ERC20ProxyDelegatorImplementation public immutable proxyDelegator;

    /**
     * ### EVENTS ###
     */

    event ProxyDeployed(address indexed delegate, address proxyAddress);
    event DelegationProcessed(address indexed from, address indexed to, uint256 amount);

    /**
     * @dev Constructor.
     * @param _token The ERC20 token address
     * @param _metadata_uri ERC1155 metadata uri
     */
    constructor(ERC20Votes _token, string memory _metadata_uri) ERC1155(_metadata_uri) {
        token = _token;
        proxyDelegator = new ERC20ProxyDelegatorImplementation(_token);
    }

    /**
     * @dev Executes the delegation transfer process for multiple source and target delegates.
     * @param sources The list of source delegates.
     * @param targets The list of target delegates.
     * @param amounts The list of amounts to deposit/withdraw.
     */
    function delegateMulti(uint256[] calldata sources, uint256[] calldata targets, uint256[] calldata amounts)
    external
    {
        _delegateMulti(sources, targets, amounts);
    }

    function _delegateMulti(uint256[] calldata sources, uint256[] calldata targets, uint256[] calldata amounts)
    internal
    {
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
        for (uint256 transferIndex = 0; transferIndex < Math.max(sourcesLength, targetsLength); transferIndex++) {
            address source = transferIndex < sourcesLength ? address(uint160(sources[transferIndex])) : address(0);
            address target = transferIndex < targetsLength ? address(uint160(targets[transferIndex])) : address(0);
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
    }

    /**
     * @dev Processes the delegation transfer between a source delegate and a target delegate.
     * @param source The source delegate from which tokens are being withdrawn.
     * @param target The target delegate to which tokens are being transferred.
     * @param amount The amount of tokens transferred between the source and target delegates.
     */
    function _processDelegation(address source, address target, uint256 amount) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }

    /**
     * @dev Reimburses any remaining source amounts back to the delegator after the delegation transfer process.
     * @param source The source delegate from which tokens are being withdrawn.
     * @param amount The amount of tokens to be withdrawn from the source delegate.
     */
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }

    function createProxyDelegatorAndTransfer(address target, uint256 amount) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
    }

    function transferBetweenDelegators(address from, address to, uint256 amount) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(from);
        address proxyAddressTo = retrieveProxyContractAddress(to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }

    function deployProxyDelegatorIfNeeded(address delegate) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(delegate);

        // check if the proxy contract has already been deployed
        uint256 bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            address userProxyDelegator = address(proxyDelegator).cloneDeterministic(bytes32(uint256(uint160(delegate)) << 96));
            ERC20ProxyDelegatorImplementation(userProxyDelegator).initialize(delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }

    function getBalanceForDelegate(address delegate) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }

    function retrieveProxyContractAddress(address _delegate) private view returns (address) {
        return address(proxyDelegator).predictDeterministicAddress(bytes32(uint256(uint160(_delegate)) << 96));
    }
}
```

Test file comparing the gas savings can be found at: https://gist.github.com/josipkoncurat/be5a290252911643cd9fc86665cfc15a.

In summary:

`deployProxyDelegatorIfNeeded(...)` function:
Current implementation: 116_468 gas
Proposed implementation: 107_050 gas

`retrieveProxyContractAddress(...)` function:
Current implementation: 5_510 gas
Proposed implementation: 1_475 gas

To transfer voting shares to two new targets:
Current implementation: 434_865 gas
Proposed implementation: 426_582 gas

To reimburse from two existing targets:
Current implementation: 150_506 gas
Proposed implementation: 138_346 gas