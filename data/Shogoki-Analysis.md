# Analysis Report

## General

I´ve spent some hours over the period of 2-3 days for this audit, covering the 2 contracts in a single file in scope. As this was a very small codebase i did expect there to be less possible attack vectors and issue.

## Approach

My approach was to first understand codebase very well. So i started to look through the code. Unfortunately there was no documentation and only a few helping comments. But nevertheless after looking at the test file, the intended usage became very clear as the code is quite straight forward. Lookin into the code of Openzeppelins ERC1155 and ERC20Votes implementation was really helpful for the overall task.

## Architecture recommendations

Overall the Architecture for this contract was quite straight forward. The intended Usage for the ERC20MultiDelegate contract was clearly to be used with the corresponding ENSToken and probably with their own frontend. However it should be clearly stated to the users of the contract how and what they have to input into the main entrance function. As it might lead to some irritation as the addresses have to be put in as uint256 values, which also leads to a minor bug, that i reported separately.

In terms of Dependency and Repo Management there are some critics, too. First of all there is an outdated version of the openzeppelin cotnracts used. Secondly it might be confusing for developers to have a package-lock.json and a yarn.lock file. These contain different Minor Versions of the openzeppelin contracts library, which would result in several bugs when using the wrong one. The readme states to use `yarn` which is good, but it would be recommended to remove the package-lock.json or to ensure that the versions are matching.

Also in the general package.json file there is a quite old version specified for the openzeppelin contracts. (^4.3.1). Even they use the carrot here, they should state the minimum (or better exact) version they require. As for example, if they would use 4.3.1 the code would not work anymore.

## Qualitative analysis

The code quality overall is pretty mature, but as beforementioned there is a bit lack of documentation. 

One important point is the usage of the ERC20Votes contract as an interface to interact with these tokens. 

First of i would recommend to use an interface instead of the full contract here.
Secondly there is a risk with non compliant (weird) ERC20 Tokens. As already stated in the Bug i reported regarding silently failing transfers, the code was written with the assumption to be used with the own ENSToken. However, the sponsor mentioned, that it should work with any ERC20 Voting token. Therefore it would be recommended to either check the returns of transfers manually, or to use a library like SafeERC20 for that.

The test file is quite complete and tests most of the invariants. Fuzz testing would be a good idea to add, as this would have probably catched some edge cases with too high uints passed in as addresses. 

My suggestion would be to add tests for the case that there are invalid uints (too high) put in to the main entrance function as well as thinking about to add Fuzz testing.

## Centralization risks

There is a minor centralization risk in regaards to the Owner of the contract. However the only functionality the owner has is to set the TokenURI of the ERC1155, whcih is not used for now.

## Time spent

I spent a total of 11 hours on 3 days for this contest


### Time spent:
12 hours