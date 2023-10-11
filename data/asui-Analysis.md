# **Overview Of multiDelegate**
The ENSToken allows voting rights to be delegated to another address but its limitation is that it can only be delegated to only one address and the full voting rights of the token holder. Either he delegate his full voting rights or not at all. This is where the multiDelegate comes in.

Multi delegate will allow users to delegate their voting rights to multiple addresses in one transaction and they can also specify how much of the rights to delegate to each address.
This is done by creating a proxy contract for every address that the ENSToken holder wants to delegate.

* **A user will interact with the multiDelegate contract to create a proxy contract which will act as the holder 
  of the token and delegate the voting rights to the address the user wants**. 

### Disadvantage
The tokens of the user will have to be transferred to the proxy contract although the main reason of delegate is to still hold the tokens and give only the voting rights to an address.
But the user can get his tokens back anytime from the **multiDelegate** contract. 

This means that the users trust the proxy and the **multiDelegate** contract to handle their tokens securely.

### The proxy contract(ERC20ProxyDelegator)
This contract will be created everytime a user wants to delegate to a new address from the **multiDelegate** contract.
* This contract will act as an address that holds the ENSToken. This way a user can create multiple proxy 
  to delegate for multiple addresses.
* This contract has no functions. 
* But the constructor will approve infinite ENStoken to the **multiDelegate** contract so that the 
  **multiDelegate** contract can handle the tokens.
* The constructor will also delegate the voting rights to the user specified address. Here the token holder 
  is the proxy contract and the user can delegate the voting rights to any address.

Because the user can specify the amount of tokens to send to the proxy and the address to delegate for that proxy address this is how a user can create multiple proxy contract for multiple address and can give any amount of voting rights. 

This is the simple proxy contract with no functions: 
```solidity
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
}
```

## Process of using the multiDelegate contract

1. When a user uses the **multiDelegate** contract for the first time he will pass the addresses he wants to 
   delegate the rights to and the amounts of ENSToken(the amount of the tokens determine the value of voting 
   rights) .
   * The contract will create a proxy contract for each address.
   * It will transfer the amounts the user pass for each address to each respective proxy contract. 
   * Now the proxy contract holds the given amount of tokens and the given address has the rights to vote for 
     those tokens.
 
This will be done by calling the ```delegateMulti``` function in the **multiDelegate** contract. No value should be passed as the sources and the targets will be the addresses the user wants to delegate to and the amounts are the amount of ENSToken for each addresses to have the voting rights.

2. When a delegated previously and wants to stop delegating and get back his tokens the user will call the ```delegateMulti``` function by passing the arguments:
   * sources - the address the user has delegated previously to.
   * targets - no value should be passed. An empty array.
   * amounts - the amount is how much of the ENSToken the user wants to stop delegating to the addresses.
In this process the specified amount of ENSToken will be sent back to the user from the respective address's proxy contract. And the voting rights of the addresses will be decreased accordingly. 

3. A user can also transfer the voting rights from one address to another directly.
  This will be done by calling the the ```delegateMulti``` function with the arguments:
       * sources - the address the user wants to move the voting rights from.
       * targets - the address the user wants to move the voting rights to.
       * amounts - how much of the rights to move from sources to targets .

In this process, the given amount of ENSToken tokens will be transfered from the sources to the targets respectively which in turn will change the amount of voting rights.
If the target address has never been delegated then a new proxy contract will be created for that target else it will send the ENSToken to the existing proxy to increase the voting rights.




Note: In the above explanation sources or targets are said to be address but it will need to be turned into uint256 first.

**The ERC1155 (multiDelegate) token:**
The ```multiDelegate.sol``` contract is also an ERC1155 token. This token represents the amount of the ENSTokens and is minted when a user delegate ENSToken(send to the proxy) to an address. 
The id of this token will be determined by the given target.
The holder of this token can get back the ENSToken by following the step 2 explained above.  




### Time spent:
48 hours