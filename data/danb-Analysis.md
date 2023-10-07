# Analysis 
## API
The current external API does not converge with the common sense a consumer will have.
This occurs since a user will not be able to send both a `reimburse` and a `createProxyDelegatorAndTransfer` "request" in the same call to the `delegateMulti` function, since the API doesn't allow for such use.

A possible situation in which the user may want to do so, is if the user had passed 100 tokens to delegatee A in the past, and they would like to reimburse them and pass 50 tokens to delegatee B.

The way the API allows the user to currently do so is to first transfer 50 of the tokens they passed to A, from A to B, and then reimburse the remaining tokens from A.
This is very unintuitive, and a much more intuitive way would be to reimburse the full 100 tokens from A and then pass 50 tokens to B.

### Time spent:
15 hours