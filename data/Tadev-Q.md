## [L‑01] Missing checks in `_delegateMulti` function for 0 values in `amounts` array 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L68

Internal `_delegateMulti` function should check that `uint256[] calldata amounts` array doesn't contain any value equal to 0. If it does, it will finally result in calling `transferfrom` with amount equal to zero, which is not a desired behavior, as it will consume gas for no action.

Recommended Mitigation Steps : 
Adding a check in the for loop could prevent executing `transferfrom` with 0 amount, and continue to the next iteration of the loop.

For example, we could add the check after this line : https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L96

```
if (amount == 0) {
  continue;
}
```


## [L‑02] A user would need to do 2 transactions to initiate a delegation with `ERC20MultiDelegate` contract

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160

In order to create a delegation for the first time using `ERC20MultiDelegate` contract, a user would need to call `delegateMulti` function, providing no source (or 0 address), one target representing the account to delegate voting power to, and one amount.

`delegateMulti` function will internally call `_delegateMulti` function, which will internally call `createProxyDelegatorAndTransfer` function. Then, the proxy is deployed if no one delegated to the specified account before, and the following line is finally executed :
`token.transferFrom(msg.sender, proxyAddress, amount)`.

This means the user first of all needs to send a transaction to the ERC20 token contract, in order to approve at least the amount specified in `delegateMulti` function inputs. This could be not user-friendly, especially if the token used in `ERC20MultiDelegate` doesn't implement ERC20permit extension. 

Impact : 
This issue could lead to poor user experience when using `ERC20MultiDelegate` to delegate voting power of a token that doesn't implement ERC20permit extension, as the Sponsors told us on the discord that `ERC20MultiDelegate` contract could be used with any ERC20 contract.

Recommended Mitigation Steps : 
Make sure the ERC20 token used within `ERC20MultiDelegate` is implementing ERC20permit properly. If not, this will result in the requirement of 2 transactions for the user to initiate a delegation.
