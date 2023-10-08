## (non-critical) ERC1155 callback function problem

`_mintBatch` and `_burnBatch` will call the ERC1155 callback function, it may cause the reentry attack. I suggest using the noreentry modifier.

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L114

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L111


## (non-critical) using `safetransferfrom`

some function transfer the token by calling `transferFrom`, I suggest using `safetransferfrom`

## (non-critical) Does not check for array length

`delegateMulti()` function does not check for array length, it may cause DoS problem and waste gas.I suggest set the Max_length