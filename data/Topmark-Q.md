### Report 1:
#### Use of safeTransferFrom over transferFrom
The codebase uses the transferFrom function for token transfers. However, it’s recommended to use safeTransferFrom instead. The safeTransferFrom function provides enhanced security by confirming that the recipient indeed received the tokens. This prevents tokens from being sent to incorrect addresses or contracts not prepared to handle them, thus enhancing the reliability of token transfers.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L148
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L160
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L170
```solidity
148.  --- token.transferFrom(proxyAddressFrom, msg.sender, amount);
      +++ token.safeTransferFrom(proxyAddressFrom, msg.sender, amount);
```
###  Report 2:
#### Missing Validation and Proper Error Handling
The contract does not have a balance validation and proper error handling before carrying out transfer from msg.sender. A proper balance validation is necessary just like it is present in [L131](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131) of same contract.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L160
```solidity
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
+++   require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
```
###  Report 3:
#### No Address validation Check after Type Conversion
The contract does not validate "source" and "target" address after type conversion from uint256 to address. This address was used to compute proxyAddress before Transfer was done between these proxyAddresses but at no point was any validation done for these addresses to ensure they are not invalid both before conversion to proxyAddress and after conversion, this could result to lose of fund due to invalid address at the various functions where they are being used.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L91
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L94
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L147
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L168-L169
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L159
###### Validation addition before conversion
```solidity
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
...
           address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
  +++ if ( transferIndex < sourcesLength ) require (source != address(0), "invalid Address")
  +++ if ( transferIndex < targetsLength ) require (target != address(0), "invalid Address")
...
```
###### Validation addition after conversion
At transferBetweenDelegators(...) function
```solidity
  function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
+++     require ( proxyAddressFrom  != address(0), "invalid Address");
+++     require ( proxyAddressTo  != address(0), "invalid Address");
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```
At _reimburse(...) function
```solidity
    function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
+++     require (proxyAddressFrom  != address(0), "invalid Address")
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```
At createProxyDelegatorAndTransfer(...) function
```solidity
   function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
+++     require (proxyAddress != address(0), "invalid Address")
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
```