
|Number |Issue  | Instances  | Amount of gas saved | 
| ----------- | ----------- | ----------- | ----------- |
| [G-01] | save value from `Math.max(sourcesLength, targetsLength)` to a local variable and reuse in other instances to prevent re computation | 1 | 141 |
| [G-02] | reduce number of times `proxyAddressTo` is calculated in `_processDelegation()` by saving the returned target proxyAdress from `deployProxyDelegatorIfNeeded` to memory | 1 | 5208 |


## [G-01] save value from `Math.max(sourcesLength, targetsLength)` to a local variable and reuse in other instances to prevent re computation. 

in `_delegateMulti()` we can get value from Math.max(sourcesLength, targetsLength) once and save it to a local varibale and use that variable in other instances where the max of sourceLength and targetslengths are re calculated. we can do it like this. 
```
   //304979 gas (before optimize),  304838 gas (after optimize) if array lengths are all 1
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
        
       uint inputsMaxLength = Math.max(sourcesLength, targetsLength);
        require(
            inputsMaxLength == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < inputsMaxLength;
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
    }

```
This way we will save about **141** gas as the initial logic costs 310187 gas while my optimizaton reduces it to 310046 gas (if all array inputs have lengths of 1). 

## [G-02] reduce number of times `proxyAddressTo` is calculated in `_processDelegation()` by saving the returned target proxyAdress from `deployProxyDelegatorIfNeeded` to memory. 

since `target` and `proxyAddressTo` are same address. In the logic flow of `_processDelegation()`, the proxy address of the target will be calculated twice via `retrieveProxyContractAddress()`. First in `deployProxyDelegatorIfNeeded()` and in `transferBetweenDelegators()`. We can reduce this calculation to one by modifying the code to save the address returned by `deployProxyDelegatorIfNeeded()` and passing it into `transferBetweenDelegators()`. This way there will be no need to recompute the proxy address for the target in `transferBetweenDelegators()`. Below is a snippet of what I propose. 

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
This way, recalculation of the target's proxyContract is reduced by one and this saves about **5208** gas from the `delegateMulti()` call. Gas cost before the optimization is 310187. Cost after is 304979. (if all array inputs have lengths of 1)
