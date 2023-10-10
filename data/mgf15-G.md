
# G1 - Remove the import `Math.sol`

>sine the `ERC20MultiDelegate` contract only use `Math.max` & `Math.min` we can use thos function in the contract for less gas costs 

here the gas cost blow :

> Before 


|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      |
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      89969  |      895546  |     525551  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4179457  |       13.9 %  | 

 
> After 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      89957  |      895525  |     525543  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4179217  |       13.9 %  | 



# G2 - Refactor require statements to save gas

> the require blow can be refactored to save gas 
```solidity
require(
            max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

to 
```diff
@@ -82,15 +77,15 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         );
 
         require(
+            targetsLength == amountsLength || sourcesLength == amountsLength,
-            Math.max(sourcesLength, targetsLength) == amountsLength,
             "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
         );
```

here the gas cost blow :

> Before 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      89957  |      895525  |     525543  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4179217  |       13.9 %  | 


> After 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      89892  |      895481  |     525483  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4179229  |       13.9 %  | 


# G3 - use unchecked block

> "unchecked" block is used to optimize gas costs 

```diff
@@ -110,10 +105,6 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
                 // Handle any remaining target amounts after the transfer process.
                 createProxyDelegatorAndTransfer(target, amount);
             }
+            unchecked {
+                ++transferIndex;
+
+            }
         }

```
# G4 - declear token as immutable 

> we can see that token is never modified after deployment and should be declared as immutable to save gas .

```diff

@@ -25,11 +25,11 @@ contract ERC20ProxyDelegator {
 contract ERC20MultiDelegate is ERC1155, Ownable {
     using Address for address;
 
+    ERC20Votes public immutable token;
-    ERC20Votes public token;
 
```
> Before 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      89892  |      895481  |     525483  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4179229  |       13.9 %  | 


> After 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  delegateMulti    |      87348  |      890625  |     522316  |           19  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4151133  |       13.8 %  | 



# G5- Use calldata instead of memory 

> Mark data types as calldata instead of memory where possible.

```diff
@@ -157,7 +148,7 @@ contract ERC20MultiDelegate is ERC1155, Ownable {
         token.transferFrom(proxyAddressFrom, msg.sender, amount);
     }
 
-    function setUri(string memory uri) external onlyOwner {
+    function setUri(string calldata uri) external onlyOwner {
         _setURI(uri);
     }
```

> Before 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  setUri           |             |              |     32670   |            1  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4151133  |       13.8 %  | 


> After 

|  Contract            |  Method           |  Min        |  Max         |  Avg        |  # calls      | 
|----------------------|-------------------|-------------|--------------|-------------|---------------|
|  ERC20MultiDelegate  |  setUri           |             |              |     32388   |            1  |
|  Deployments         |                   |          -  |           -  |        -    |  % of limit   |
|  ERC20MultiDelegate  |                   |          -  |           -  |    4148707  |       13.8 %  | 

