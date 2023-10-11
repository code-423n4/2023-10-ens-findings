## Low[1] - Check for the zero inputs in the target array, source array and amount array.
Source array is passed into the function `delegateMulti` but there is no check if some value passed in that array is actually zero address. So essentially this leads to delegating the some amount to the zero address after deploying the proxy for it, which is completely unnecessary and useless, Even if done by mistake and can be recovered there is no point of doing that and letting tokens sitting idle in the proxy contract.

Add the check for zero address value for the target array passed.

Similarly if there is some zero value in the source array, it may try to transfer the tokens from the no deployed proxy or it may be deployed as previously discussed if at some point someone passed zero address in target array. But will eventually lead to unexpected reverts in `_processDelegation` and `_reimburse`

Similarly there should be zero address check for the values passed in the `amounts[]` array, or safeTransferFrom will be called with the 0 amount values, and similarly _mintBatch could be called for the 0 amount for a particular if.

```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63

## Low[2] - Add the check for the passed metadataUri is not an empty array
Add the check in the `ERC20MultiDelegate.sol` for the empty string in the constructor while passing its value to the ERC1155 constructor.

```solidity
    constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) {
        token = _token;
    }
```


## Low[3] - When deployed the new proxy check if it codesize is not zero and deployment happened succesfully
`deployProxyDelegatorIfNeeded` function deploys the proxy if it is already not deployed, but it does not check the return value, if it was actually deployed.

```solidity
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
```

If for some reason the deployment fails, following can happen:

- if the proxy is not deployed we return the derive address. This function is called in `createProxyDelegatorAndTransfer` which first deploy the proxy and than transfer the tokens from msg.sender to the proxy address which we assumed have approved this address for max tokens. But a failed deployment means no approval which means the token sent on the following lines would be permanently lost, as the deployment on the address failed.

```solidity
   function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        // @note - this transfer will lead to loss of funds if the deployment failed in above step, as there is no code size check for new deployment
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
```

change the `deployProxyDelegatorIfNeeded` as follow
```diff
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        if (bytecodeSize == 0) {
-            new ERC20ProxyDelegator{salt: 0}(token, delegate);
+            address proxyAddress = new ERC20ProxyDelegator{salt: 0}(token, delegate);
+            require(!isZero(proxyAddress),"deployment failed"); 
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```
## Low[4] Cap the size of arrays passed in `delegateMulti` function to prevent out of gas error
The functions takes 3 arrays as input, cap the size of these to a reasonable size in first lines to prevent the out of gas error while looping later on in the function to save cost and avoid unexpected reverting.

Cap the following arrays in function:
```solidity
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63


## NC[1] Use the latest version of openzeppelin library for optimal performance and least bugs
Openzeppelin recently released the new version of their library, version 0.5 is the first major release of the Solidity library since 2021. This version comes with many optimisation in terms of gas cost, removing the bad/redundant and non useful pieces and have few bugs removed too from the previous versions. 

Some of the main upgrades are:
1. Namespaced Storage for better upgrades security
2. Leveraging new Solidity compiler additions
3. Tooling updates
4. Gas-efficiency without compromising Security
5. Replacing revert strings with custom errors
6. Removing duplicated SLOADs and adding immutable variables
7. Packing variables in storage
8. Flexible and Transparent Access Management
9. Redefining Access Control through AccessManager
10. Best-in-class Audit & Testing Practices

So instead of using the old version, the project is using ^4.3.1. Replace the old version with newer one to get advantage of all these new updates.