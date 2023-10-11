##  <img src="https://user-images.githubusercontent.com/68193826/274388942-4cdcfee3-3a30-405e-8dda-1506cb1802af.png" loading="lazy" width="30" alt="" class="image-9"> Description
ENS (Ethereum Naming Service) is a distributed, open, and extensible naming system based on the Ethereum blockchain. ENS provides a way to attach the identiy to a protocol. Ens have their own token name ENS - Yes very intuitive :)

This audit is more focused on their new product, where the ENS token that is an ERC20 extension ERC20 votes that lets to delegate the votes to another address. Multi Delegate contract allows to delegate to multiple addresses in one transaction, move the funds between the proxies and reimburse the funds back to real owner, all done via the erc1155 nfts, really different approach from other voting delegation based protocols I audited before.

**The key contracts of the protocol for this Audit are**:

- **ERC20MultiDelegate.sol**: 
This contract have main one function which:
1. Takes arrays of sources, so source is actually the address of the delegate and we derive the proxy address from this address using the predefined formula of create 2.
2. Second is target address, which works also similar to the target.
3. Third is the amount array which defines the amounts that will be moved from proxy to msg.sender, from source to target or from msg.sender to proxy.

We have five routes:

1. Sources and Targets have the same length, in that case we transfer the tokens from one proxy to other, each proxy delgating the tokens to a specific target address. So we burn the nfts of source id and mint the nfts with target id's to the msg.sender.

2. Sources array is passed as empty and we have something in the target, in that case tokens are transferred from the msg.sender to the target proxy address derived, if deployed already than it is used, otherwise for a new target, new proxy is deployed that delegates the tokens. And we mint the nfts to user to the amount sent to target with the id of target address

3. Target array is empty but sources array is not, in this case source is address of the target (yeah confusing first time) to which previously msg.sender delegated. In this case tokens from that proxy are transferred back to msg.sender and we burn the same amount of nft from the user with the id source address.

4. Source array length < target array length — For indexes where in both source and target exist, _processDelegation is called, for rest of the target length indexes createProxyDelegatorAndTransfer is called.

5.Source array length > target array length — For indexes where in both source and target exist, _processDelegation is called, for rest of the source length indexes _reimburse is called.


**Existing Patterns**: The protocol uses standard Solidity and Ethereum patterns. It uses the `ERC1155` `ERC20` and `ERC20Votes` extension for most part and `Ownable` pattern to grant the ownership to the deployer.

##  <img src="https://user-images.githubusercontent.com/68193826/274388942-4cdcfee3-3a30-405e-8dda-1506cb1802af.png" loading="lazy" width="30" alt="" class="image-9"> Approach
I took the top to down approach for this audit, first having a general look at all the function and than from main external function to the internal and private function that are called with in the other functions, so general flow is as follow

```
1. delegateMulti
2. _delegateMult
3. _processDelegation
4. _reimburse
5. transferBetweenDelegators
6. deployProxyDelegatorIfNeeded
7. retrieveProxyContractAddress
8. getBalanceForDelegate
```
`deployProxyDelegatorIfNeeded` and `retrieveProxyContractAddress` were viewed side by side as they are pretty interlined. The protocol is dependent upon the deterministic approach of deployment where using the salt as zero consistently as there is no negative impact of it so it is safe to use with 0 salt.

##  <img src="https://user-images.githubusercontent.com/68193826/274388942-4cdcfee3-3a30-405e-8dda-1506cb1802af.png" loading="lazy" width="30" alt="" class="image-9"> Codebase Quality
1. The codebase maintains a high level of quality; however, a significant drawback is the absence of comprehensive documentation.
2. Inadequate documentation posed challenges when attempting to grasp the intricacies of the _multiDelegate function and its multiple routes.
3. The need arises for comprehensive documentation to elucidate the rationale behind proxy deployment, the timing and purpose of NFT minting, as well as the mechanics of reclamation and proxy-to-proxy transfers.
4. By providing clear and detailed documentation alongside thorough comments, we can not only enhance the codebase's auditability but also make it more accessible to users with limited experience.


| Codebase Quality Categories  | Comments |
| --- | --- |
| **Unit Testing**  | Code base is well-tested but there is still place to add fuzz testing and invariant testing and foundry have now solid support for those, I would say best in class.|
| **Code Comments**  | Overall comments in code base were not enough, more commenting can be done to more specifically describe specific scenerios. At many point if there were more detail commenting and natspec it would have been bit easy for the auditor and less question would have been asked from sponser, saving time for both. 
| **Documentation** | Documentation was not detailed enough, such novelty requires more documentation than one paragraph, that can be improved and I am sure will be done as previous products have full proof documentation on enswebsite.|

##  <img src="https://user-images.githubusercontent.com/68193826/274388942-4cdcfee3-3a30-405e-8dda-1506cb1802af.png" loading="lazy" width="30" alt="" class="image-9"> Systematic and Centralisation Risks

### 1. Centralization Risk: 

In the code there is only one function that is owner controller `setURI`.

So there is no centralisation risk in the code base, it seems really permissiona-less following the ethos of blockchain decentralisation.

### Time spent:
10 hours

### Time spent:
10 hours