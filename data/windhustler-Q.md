### Remove unused Address library
[Address library](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L26) is not used anywhere in the code. 
Remove the using `Address for address` and the [import statement](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L4).

### Consider using the Address library for isContract check
```solidity
        address proxyAddress = retrieveProxyContractAddress(token, delegate);
        // check if the proxy contract has already been deployed
        uint256 bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            //            uint256 gas = gasleft();
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
```

can be simplified to:

```solidity
        address proxyAddress = retrieveProxyContractAddress(token, delegate);
        if (!proxyAddress.isContract()) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
```

### Cache Math.max(sourcesLength, targetsLength) in a variable
`Math.max(sourcesLength, targetsLength)` is used for comparison with the [`amountsLength`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79-L82) and inside the [for loop](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87).
It can be cached in a variable to avoid recomputation on each for loop round.
```solidity
uint256 max = Math.max(sourcesLength, targetsLength);
```

### Replace OpenZeppelin ERC1155 with Solmate ERC1155
Consider replacing the OpenZeppelin ERC1155 contract with the more lighweight [Solmate ERC1155](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC1155.sol).

### Skip the amount if it is zero
If the [amount passed](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L96) is 0 skip the iteration.
```solidity
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
            if (amount == 0) {
                continue;
            }
```

### Consider replacing `ERC20ProxyDelegator` with something more lightweight

For each new user a `ERC20ProxyDelegator` contract is once deployed. This contract is used to store the `ERC20Votes` token on user behalf.
This contract has no functions, and it only serves as a delegate for the user. That being said the runtime and creation code of this contract should be as minimal as possible. 
The `ERC20ProxyDelegator` has a creation code (`type(ERC20ProxyDelegator).creationCode.length`) of 460 bytes and runtime code (`type(ERC20ProxyDelegator).runtimeCode.length`) of 63 bytes.
This adds to the cost of the deployment of the `ERC20ProxyDelegator` contract that is incurred for each new address entering the system.

Consider replacing the `ERC20ProxyDelegator` with a more lightweight contract that would serve the same purpose. I've already outlined the usage of EIP-1167 Minimal Proxy as a delegate in my gas report.
The Clone deployed for each new user is only 55 bytes large(45 bytes for the runtime).

Alternative `ERC20ProxyDelegator` would be:

```solidity
contract ERC20ProxyDelegator{
    ERC20Votes internal immutable token;
    address internal immutable owner;

    function initialize(address _delegate) external {
        // This is only going to be called once since the owner is the ERC20MultiDelegate contract. So no need for initializer.
        require(msg.sender == owner);
        token.delegate(_delegate);
        token.approve(msg.sender, type(uint256).max);
    }

    constructor(ERC20Votes _token) {
        token = _token;
        owner = msg.sender;
    }
}
```
 This would serve as the implementation, and a [clone using openzeppelin Clones lib](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/Clones.sol) pointing to this implementation would be deployed and initialized for each new user.
```
import "@openzeppelin/proxy/Clones.sol";

contract ERC20MultiDelegateProxyClones is ERC1155, Ownable {
    using Clones for address;

    constructor(ERC20Votes _token, string memory _metadata_uri) ERC1155(_metadata_uri) {
        token = _token;
        proxyDelegator = new ERC20ProxyDelegator(_token);
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
            uint256 gas = gasleft();
            address userProxyDelegator =
                address(proxyDelegator).cloneDeterministic(bytes32(uint256(uint160(delegate)) << 96));
            ERC20ProxyDelegatorImplementation(userProxyDelegator).initialize(delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
        
    function retrieveProxyContractAddress(address _delegate) private view returns (address) {
        return address(proxyDelegator).predictDeterministicAddress(bytes32(uint256(uint160(_delegate)) << 96));
    }
}
```

Whereby the `proxyDelegator` is the implementation contract deployed once during the `ERC20MultiDelegate` deployment.

### Check if the source and target are the same addresses
Insert a check to determine if [`proxyAddressFrom` and `proxyAddressTo`](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163-L171) are the same address. 
If they are skip the transfer.
```solidity
        address proxyAddressFrom = retrieveProxyContractAddress(from);
        address proxyAddressTo = retrieveProxyContractAddress(to);
        if (proxyAddressFrom == proxyAddressTo) {
            return;
        }
```

### Add a zero address check for each target
User can set the target as `address(0)` in which case the `ERC20ProxyDelegator` will get deployed for a delegatee being zero address.
The impact of this is that the deployed `ERC20ProxyDelegator` will be holding the `ERC20Votes` tokens but there will be no actual voting power attributed to it. 
This is due to the fact how [`Votes.sol` contract handles moving delegate votes in case of a zero address](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/governance/utils/Votes.sol#L193).
The issue with no votes attributed can be checked through the following test:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import {Test} from "forge-std/Test.sol";

import {ENSToken} from "./ENSToken.sol";

import "../src/ERC20MultiDelegate.sol";

import "@openzeppelin//token/ERC1155/utils/ERC1155Holder.sol";

contract Deployer is ERC1155Holder {}

contract ERC20MultiDelegateTest is Test {
    ERC20MultiDelegate private multiDelegate;
    address private votesToken;
    Deployer private deployer;
    Deployer private deployer2;

    address private source1 = makeAddr("source1");
    address private source2 = makeAddr("source2");

    address private target1 = makeAddr("target1");
    address private target2 = makeAddr("target2");

    uint256 private amount = 1e18;
    
    function setUp() public {
        deployer = new Deployer();
        deployer2 = new Deployer();

        vm.startPrank(address(deployer));
        votesToken = address(new ENSToken(100e18, 100_000e18, block.timestamp));
        ENSToken(votesToken).transfer(address(deployer2), 50e18);

        multiDelegate = new ERC20MultiDelegate(ERC20Votes(votesToken), "http://localhost:8081");

        ENSToken(votesToken).approve(address(multiDelegate), type(uint128).max);
        vm.stopPrank();
    }
    function testTargetToAddressZero() public {
        uint256[] memory sources = new uint256[](0);

        uint256[] memory targets = new uint256[](1);
        targets[0] = uint256(uint160(address(0)));

        uint256[] memory amounts = new uint256[](1);
        amounts[0] = amount;

        vm.startPrank(address(deployer));

        multiDelegate.delegateMulti(sources, targets, amounts);

        vm.stopPrank();

        amounts[0] = 1;

        vm.startPrank(address(deployer2));
        ENSToken(votesToken).approve(address(multiDelegate), type(uint128).max);
        multiDelegate.delegateMulti(sources, targets, amounts);
        vm.stopPrank();

        console.log("balance deployer", multiDelegate.balanceOf(address(deployer), uint256(uint160(address(0)))));
        console.log("balance deployer2", multiDelegate.balanceOf(address(deployer2), uint256(uint160(address(0)))));

        address proxyAddress = multiDelegate.retrieveProxyContractAddress(ENSToken(votesToken), address(0));
        console.log("proxyAddress", proxyAddress);

        console.log(ENSToken(votesToken).balanceOf(proxyAddress));

        console.log(ERC20Votes(votesToken).delegates(proxyAddress)); // This is address(0)

        console.log(ERC20Votes(votesToken).getVotes(address(0))); // This is 0, although in case of any other address it would be the value of ENS token on the proxy address
    }
}
```