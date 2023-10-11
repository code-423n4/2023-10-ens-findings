
## Missing address(0) check in `deployProxyDelegatorIfNeeded`

## Description
The `deployProxyDelegatorIfNeeded` retrieves the contract address of the delegator contract, however it does not check wether the returned address is address(0), although unlikely if the returned address turns out to be address(0), the tokens delegated will be burned in the `createProxyDelegatorAndTransfer`.
```
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
   @>   address proxyAddress = retrieveProxyContractAddress(token, delegate);
            ...
    }

    function createProxyDelegatorAndTransfer( 
        address target,
        uint256 amountcreateProxyDelegatorAndTransfer
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
  @>    token.transferFrom(msg.sender, proxyAddress, amount);
    }
```

## Links
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L173-L183

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161

## Fix

Use a require statement to prevent the usage of address(0) such as :
```
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);
        require(proxyAddress != address(0), "Proxy address cannot be 0x0");
```

## Recommended usage of OpenZeppelin v5.0.0 contracts 
## Description
Main contract imports both ownable  and 1155 contracts which have been updated. 1155 has been modified to be more efficient. 
Ownable has also incurred some slight modifications: "Prevent using address(0) as the initial owner. && Added an initialOwner parameter to the constructor, making the ownership initialization explicit."
The usage of these new features would make the contract more readable , optimized and up to date to the current DeFi landscape. 

