## QA Report 


### [Q-1] Unused library Address.
The contract ERC20MultiDelegate imports the library 'Address', but does not use it. It is recommended to remove the library import and 'using Address for address;' for code purity.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L26

### [Q-2] Unnecessary assignment of zero address.
No need in line 92 and 95 assign address(0) to variable, because it these cases, these values do not use further down the code. This design can be replaced with a regular IF without an 'else' block.
```diff
-    address source = transferIndex < sourcesLength
-                ? address(uint160(sources[transferIndex]))
-                : address(0);
-    address target = transferIndex < targetsLength
-                ? address(uint160(targets[transferIndex]))
-                : address(0);
+    address source;
+    address target;
+    if(transferIndex < sourcesLength) {
+                source = address(uint160(sources[transferIndex]));
+    }
+    if(transferIndex < targetsLength) {
+                target = address(uint160(targets[transferIndex]));
+    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L90-L95

### [Q-3] Unnecessary balance check.
There is no need to check additionally the balance in the _processDelegation() function, because this point is checked in the token burning function - _burnBatch(). If the user specifies a larger balance than he delegated, the contract will not be able to burn more tokens than he has (ERC-1155).

We can delete assert() and function getBalanceForDelegate(). So, it will decrease deploy cost.
```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
-        uint256 balance = getBalanceForDelegate(source);

-        assert(amount <= balance);


-    function getBalanceForDelegate(
-        address delegate
-    ) internal view returns (uint256) {
-        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
-    }
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129-L131

### [Q-4] Unnecessary event ProxyDeployed.
There is no point in emitting an event 'ProxyDeployed', because users do not directly interact with the proxy contract(ERC20ProxyDelegator) and they do not need its address. If there is no proxy for the target address, it is created without user intervention and the user does not need to care whether there is a proxy for the target address or not. This information does not carry any semantic load.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L187C18-L187C31

### [Q-5] The user does not know how many more tokens can be delegated.
In the ERC20Votes from Openzeppelin contract there is a restriction, that the maximum delegation amount can be type(uint208).max. But the contract accepts the amount for delegation as uint256. If the user specifies an amount greater than type(uint208).max, then the transaction will be reverted. 
If someone has already delegated their tokens to the address, then the user needs to know the maximum number of his tokens he can delegate (up to type(uint208).max). 
Recommended to create a view function that will calculate the available number of tokens for delegation to the specified address. 
```solidity
function allowedDelegate(address target) external view return(uint) {
    return type(uint208).max - token.balanceOf(retrieveProxyContractAddress(token, target);
}
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/793d92a3331538d126033cbacb1ee5b8a7d95adc/contracts/token/ERC20/extensions/ERC20Votes.sol#L30
