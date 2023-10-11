### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Centralization risks 
|d) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |\
|e) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

I began by examining the scope of the code, which guided my approach to reviewing and analyzing the project.
I analyzed and audited the codebase following the details below;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-ens#tests)|Test and installation structure is simple, cleanly designed|

## b) Analysis of the code
A simple and straightforward codebase that enables votes to be delegated to target. It inherits `ERC 1155` to be able to keep track of all delegates for a token holder. Once a token holder makes a call to `delegateMultifunction` to delegate their token, a proxy will be deployed for each of the delegate that the token holder has decided to delegate to. 

## c) Centralization risks 
There is only one risk in the controlled scope. 

```solidity
    function setUri(string memory uri) external onlyOwner {
      _setURI(uri);
``` 

## d) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding. Below are some more recommendations:

**Gas Optimization Recommendations**
Amounts should be checked for 0 before calling a transfer
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L160C1-L160C62
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L170C8-L171C6
 ```solidity
 token.transferFrom(msg.sender, proxyAddress, amount);
    }
```
```solidity
 token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
```

## e) New insights and learning from this audit 

- I learned in depth, the usage and details of the `ERC 1155`  contract.



### Time spent:

 Time Spent ⏱
I spent a total of `2 days` fully dedicated to completing this audit, distributed as follows:

1. 1st Day: I took my time trying to comprehend the protocol's functionalities and implementation. I also ran some tests that were provided by ENS project team.
2. 2nd Day: Focused on conducting a thorough analysis, derived from the contracts and information provided by the protocol.

48 hours

### Time spent:
48 hours