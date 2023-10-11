## Intro

The focus of this audit contest was the `ERC20MultiDelegate` contract which is used by a delegator to delegate their voting power to multiple addresses in a single transaction. 

## Analysis of the codebase

Even though it is supposed to be a part of the greater ENS governance system, its dependence on other external libraries is minimal. It's only dependent on a few popular OZ libraries like `ERC1155`, `ERC20Votes`, `Math` which makes it easy to understand even without much context. Also, the fact that the codebase is just 200 lines, makes the codebase concise and straightforward.

So, there are 3 main players in this system. The delegator, the source delegate and the target delegate. There are a few scenarios involving these 3 players based on the length of the `sources` and `targets` inputs array in the `delegateMulti` function: 

1. Initially the delegator only wants to delegate his votes to his preferred delegates. For this, the delegator uses `delegateMulti` function and passes only the `target` array and an empty `source` array. This would simply transfer the delegator’s tokens to the `ERC20ProxyDelegator`  associated with the delegates(target) and then delegate the votes/tokens to the target addresses.

2. If `sources` and `targets` of equal length are passed, then the tokens are transferred from one proxy (`ERC20ProxyDelegator`) to another, each proxy delegating the tokens to the target address associated with them.

3. If `sources` are not empty and `target` addresses are empty, then that means the delegator wants to send the tokens that they earlier delegated to the `source` back to themselves.

One very interesting aspect of this contract is the way it uses ERC1155 tokens. An ERC1155 token is minted whenever a delegator gives their voting power to a particular delegate. They are used in a way to keep track of delegated tokens actually owned by the delegator, in all different proxy delegate contracts. So, if a delegator wants to delegate 1000 tokens to address A, 1000 ERC1155 tokens are minted to the delegator. These tokens will have a unique `id` which equals `uint256(uint160(address of A))`. In this way, a distinct ID is associated with each delegate's address.

Just as ERC1155 tokens are minted when a user delegates their token to a delegate, they are also burned when the user wants to transfer their voting power from that specific delegate to another delegate. In that case, the ERC1155 tokens, associated with the delegate from whom the votes are being transferred are burnt and ERC1155 tokens associated with the delegate to whom the votes are being sent to are minted.

## Centralisation risks

The centralisation risks associated with the codebase are of no major significance. The only place where the `onlyOwner` modifier is used is in the `setUri` function. This is actually required to be like that. Only the owner of the contract should be expected to change the URI for the ERC1155 contract. Even if the owner was malicious, there would be no harm to the way the core functionality of the contract works.

## Recommendation

The codebase is pretty solid. There don't seem to be any major risks associated with the contract. But, some modifications can be made to decrease the gas used in the contract. For example, newer solidity versions like 0.8.19 can be used. Simple things like putting the iterator of a for loop in an unchecked block, and using `++transferIndex` instead of `transferIndex++`, to name a few.

### Time spent:
10 hours