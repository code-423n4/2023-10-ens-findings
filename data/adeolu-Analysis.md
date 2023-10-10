## Approach

The repo for this audit has just one file in scope which is 216 lines. The file in scope contains `ERC20MultiDelegate` and `ERC20ProxyDelegator` contracts. The `ERC20MultiDelegate` also uses the industry standard openzeppelin library in its implementation/extension of ERC1155, ERC20Votes and Ownable contracts. This `ERC20MultiDelegate` is developed to be used in ENS governance processes. 

I first took a first read to understand the development specs /mindset of the developer of this `ERC20MultiDelegate` contract and how it interacts with the `ERC20ProxyDelegator` and the external `ERC20Votes` token. 

From my understanding, the `ERC20MultiDelegate` functionality is to let ENS delegators delegate their votes/voting power/votes token to another address (delegatee) or transfer it from one delegatee to another delegatee. Each new delegatee address is given a `ERC20ProxyDelegator` contract which is deployed on behalf of the delegatee. 

Every delegation is done via the [delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57) fcn primarily involves three main state changes: 
-  transfer of a specified amount of the ERC20Votes token to the target delegator address's `ERC20ProxyDelegator` contract
-  mint's a specified amount of ERC20MultiDelegate erc1155 tokens to the delegator, with the id or ids specified as the delegatee/target address. 
-  burns a specified amount of ERC20MultiDelegate erc1155 tokens from the delegator, with the id or ids specified as the source address. 

[delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57) fcn here is external and not gated. This is the main user entry point. 

The noted invariants are: 
- The amount of `ERC20Votes` token with a delegate address's `ERC20ProxyDelegator` contract is supposed to be equal to the `ERC20MultiDelegate` erc1155 token balance of the delegator for the token type which has an ID of delegate address. 

- During delegation vote transfer, the delegator should decrease their balance of `ERC20MultiDelegate` ERC1155 tokens which have their ID as the source delegate by using `_burnBatch()` for the specified amount. Simultaneously, they should increase their balance of `ERC20MultiDelegate` ERC1155 tokens which have their ID as the target delegate by using `_mintBatch()` for the same amount.

## Architecture recommendations
Here the major optimization i suggest is in the `_processDelegation()` fcn. `transferBetweenDelegators()` can be modified so we can pass the `proxyAddress` returned from the `deployProxyDelegatorIfNeeded`() into `transferBetweenDelegators()` and use that `proxyAddress` for `proxyAddressTo`  in the  `transferBetweenDelegators()` instead of calling `retrieveProxyContractAddress()` twice for `proxyAddressFrom` and `proxyAddressTo`.  This will reduce the number of `retrieveProxyContractAddress()` calls in the `_processDelegation()` logic flow to 2 from 3 (because `retrieveProxyContractAddress()` is also called in `deployProxyDelegatorIfNeeded()` fcn). This will the reduce computation. 

Here's an example snippet of what i am proposing. 
```
    function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) public view returns (address) { //changed from private to public
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

    function transferBetweenDelegators(
        address from,
        address to,
        address proxyOfTarget,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo;
        if (proxyOfTarget == address(0)) {
        // means proxyOfTarget not passed in or set or known. we may now compute it
           proxyAddressTo = retrieveProxyContractAddress(token, to);
        } else {
            proxyAddressTo = proxyOfTarget;
        }
        
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }


    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        address proxyOfTarget = deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, proxyOfTarget, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
This should save about 5208 gas. 

## Codebase quality analysis
Overall the code in scope is of good quality, but not of highest quality as i noticed it doesnt follow all the solidity style guide rules i.e the functions are not arranged in the correct manner (layout), there is an external function in between two internal functions. Also not all functions have well detailed comments describing what they do. i.e `createProxyDelegatorAndTransfer()` and `transferBetweenDelegators()`.  These functions may be hard to understand for less skilled devs/ users. 

## Centralization risks
The contract has centralization risks as it is ownable and the owner is a single address entity. This is further corrobarated by the bot report issues [M-O1](https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#m01-centralization-risk-for-privileged-functions) and [L-01](https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#l01-use-ownable2step-instead-of-ownable).






### Time spent:
30 hours