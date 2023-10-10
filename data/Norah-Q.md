## Consider Non-reentrant Modifier for delegateMulti()

 - Protocol uses ERC1155 tokens to track the amount of tokens delegated by delegators to various delegates.

 - ERC1155 has the callback() hook, which gets executed whenever user any tokens are recieved.

 - As of now there does not seem any possibility of attack vector as the code follows change-affect-integration pattern, but still it would be best to implement non-reentrant modifier as extra measure against the any potential reentrancy attack.