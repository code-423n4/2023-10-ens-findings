## Introduction
The Multi Delegate contract is a solution that uses Proxy Contracts to enable a user to delegate voting power to more than 1 address and then further manage the delegation of the votes. Multi Delegate also inherits ERC1155 as a tool to keep track of the user's votes that are being distributed to the Proxy contracts & enable user's to redeem their tokens.  
## Technical Review
The contract has 2 external functions - `setUri()` - which is used only by the admin and a second external function - `delegateMulti()` - which is the single entry point for users. The summarized functionality of the `delegateMulti()` function would be
1. Create a Proxy (if not already created) & transfer ERC20 Vote tokens for delegation
2. Transfer owned ERC20 Vote tokens in-between Proxy Contracts
3. Withdraw ERC20 Vote tokens

### Creating Proxy contracts & transfering tokens
1. User inputs only target addresses & amounts
2. If there isn't an existing Proxy for the target address - create one
3. Proxy does an infinite approval to Multi Delegate contract & delegates to the target
4. Multi Delegate transfers user's ERC20 Vote tokens to the Proxy
5. Multi Delegate mints ERC1155 token's to the user corresponding to his deposit in the Proxy

### Transfer ERC20 Vote tokens between Proxys
1. User inputs a source, a target & amount
2. Deploy target Proxy (if needed)
3. Check if user owns ERC1155 tokens corresponding to the amount of ERC20 Votes to be moved from the source
4. Multi Delegate transfers ERC20 Vote tokens from source Proxy to target Proxy
5. Multi Delegate burns ERC1155 tokens from the user corresponding to the source
6. Multi Delegate mints ERC1155 tokens to the user corresponding to the new tokens in the target Proxy

### Withdrawal of ERC20 Vote tokens
1. User inputs only source addresses & amounts
2. MultiDelegate transfers tokens from source proxy to the user
3. MultiDelegate attempts to burn ERC1155 tokens corresponding to the withdrawn amount from the source
4. If burning reverts - user didn't have enough ownership tokens to withdraw the input amount

## Areas of concern
The Multi Delegate contract is implemented in a very secure way. The only concerns would be third parties integrating with it & understanding the mechanism of burning & minting. Because of how burning and minting is implemented it is not possible in a single transaction to move an amount of ERC20 Vote tokens through several proxies. 

## Centralization risks
The only admin operation is `setUri()`. The contract doesn't pose any centralization concerns. 

## Conclusions
No vulnerabilities were found. 

### Time spent:
12 hours