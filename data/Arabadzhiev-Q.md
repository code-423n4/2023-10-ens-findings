# [L-01] The `ERC20MultiDelegate` is using the `Address` library for `address` variables, but none of its functions are used anywhere

The functions of the `Address` library are not used anywhere inside of the `ERC20MultiDelegate`, even though it is being used for `address` variables. Either make use of the library where originally intended, or remove the `using` statement for it + its import.

## Lines of code

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L26

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4

# [N-01] Event emissions are missing after key actions within the `ERC20MultiDelegate` contract

The contract emits an event after a re-deposit takes place, but it does not do so after a deposit or withdrawal does. Additionaly it does not emit an event after the ERC1155 URI is changed. It is generraly recomended to emit meaningfull events after all key state variable updates take place, in order to facilitate easier off-chain monitoring.

## Lines of code

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144-L149

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151-L153
