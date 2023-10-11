# L-01 : Lack of input validation for targets .  
The `Multidelegate` contract takes input of addresses as uint256 which is quite unique . Each target address is inputted as uint256 and casted down to uint160 .
```solidity 
 for ( 
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex])) //IF array ends then source address will be zero 
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex])) //IF array ends then target  address will be zero 
                : address(0);
            uint256 amount = amounts[transferIndex]; 
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L94
And ERC1155's `_mintBatch`  & `_burnBatch` is used to ensure access control . Where sources/targets refer to unique tokenID of the proxydelegator .
```solidity 
 _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
 _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
```

It is could be assumed that totalsupply of a tokenID(sources/targets here) in  `ERC20MultiDelegate` contract is equal to the voting token  amount hold by the proxyDelegator . 

In this design ensuring the validity of input solely depends on the user .  However this may get problematic if sources/targets array input is bigger than `type(160).max `,  which is invalid  . There are no checks to handle this scenario . In this case,  the transaction will still execute succesfully if the user is trying to delegate . 

Other impacts of this scenario  could be : 
 The casted down uint160 from larger uint256 may match with existing delegatee and relevent proxydelegator  contract . In that case , funds will be sent to that proxydelegator if the user is trying to delegate . ERC1155 will mint tokens to the user where tokenID will be the non-casted uint256 . Now , two different tokenID in the `ERC20MultiDelegate` contract referring to the same proxydelegator contract  . 
This breaks the invariant of totalsupply(of a tokenID in `ERC20MultiDelegate` contract ) to amount of tokens hold by the delegatee's proxydelegator . 

Also ,There's risk of  a delegatee  using  those Mistakenly sent tokens before the delegator claims them back by frontrunning  .   
# recommendation :
The mitigation is pretty simple : 
All of the input array for addresses should be type of `uint160[] ` .



# N-01 : No balance check before calling token transfer in `_reimburse` function .  
It is best practice to check the balance before transferring funds . But `_reimburse` function doesnot check balance of the proxyDelegator before calling transferfrom. Although token transfer will fail in that case but it's always ideal to check balance of the proxydelegator and throw an error to reduce confusion why transfer failed .  

```solidity 
function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source); //@ci low no balance check ??  altho it'll fail at the below line . 
        token.transferFrom(proxyAddressFrom, msg.sender, amount); 
    }

```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144
# recommendation :
Rewrite the funciton as below: 
```solidity 
function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
+       require (amount <= token.balanceof(proxyAddressFrom) , "invalid amount " );
        token.transferFrom(proxyAddressFrom, msg.sender, amount); 
    }

```

# N-02: Use `amountsLength` instead of `Math.max(sourcesLength, targetsLength)` to reduce code complexity 
`amountsLength` and  `Math.max(sourcesLength, targetsLength)` the exactly same value . So, it's better to use `amountsLength` instead of `Math.max(sourcesLength, targetsLength) ` inside for loop for cleaner code . 
```solidity 
 require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80

# recommendation :
Re-write it as below : 
```solidity 
  for (
            uint transferIndex = 0;
            transferIndex < amountsLength ;
            transferIndex++
        )

```
# N-03: Lack of overall documentation . 
 The `ERC20MultiDelegate.sol` lacks in-line documentation . Some functions do have in-line docs but most of the functions lacks proper docs . 
