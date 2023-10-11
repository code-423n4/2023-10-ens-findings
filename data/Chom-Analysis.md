## Any comments for the judge to contextualize your findings

Mostly related to token transfer and synchronization between the actual ERC20 balance and the ERC1155 balance.

## Approach taken in evaluating the codebase

1. Read the instructions and the scope.
2. Skimming through the code to understand high-level views.
3. Skimming through the test cases.
4. Thinking of contract dependencies that are unlikely to be covered in the test cases.
5. Thinking of edge cases that aren't covered in the test cases primarily from uncovered contract dependencies.
6. Proof if these edge case findings are valid.
7. Write the final report and submit.

## Architecture recommendations

* Split ERC20ProxyDelegator and ERC20MultiDelegate into two different files.
* ERC20MultiDelegate should implement IERC20MultiDelegate where developers can import for use in their code without having to import an entire contract.
* ERC20MultiDelegate should override ERC165 supportsInterface to support IERC20MultiDelegate.
* Publish ENSToken and its dependencies to the NPM registry and import them instead of copying files to the repo.
* Add more test cases on ERC1155 behaviors.
* Should publish this repo to the NPM registry to enable other developers to install and import to their code.

## Codebase quality analysis

Acceptable quality. 
* Variable names make sense.
* Break into multiple functions with sensible names.
* Logic has commented to explain what is going on.

Negative factors
* It would be better if you put the NatSpec document on all functions.
* One file contains multiple contracts.
* ENSToken code is from a different module but is being copied to the repo instead of imported.

## Centralization risks
* ERC20MultiDelegate is Ownable
* Only the owner can set the URI of the ERC1155 token

## Mechanism review

Implementing multi delegation system this way may cause some problems for those who want to keep ENS tokens in their wallet for discord role/airdrop opportunity/etc. As it forces the user to transfer ENS tokens to newly deployed proxy contracts in order to do multiple delegations.

## Systemic risks
* Syncing of the actual proxy balance and ERC1155 token balance.
* ERC20MultiDelegate may not be supported by some smart contract wallets due to the lack of IERC1155Receiver implementation.

### Time spent:
10 hours