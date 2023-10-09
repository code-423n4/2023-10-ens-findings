For this ENS contest, one contract, delegateMulti, was scoped.
It was a well-structured contract.

## Code Structure
`ERC20MultiDelegate` is a contract that inherits from `ERC1155` and has two entry points.
* delegateMulti
* setUri

`delegateMulti` takes a uint256 array of three arguments: `sources`, `targets`, and `amounts`.
It is responsible for delegating from source to target by amount.

`setUri` is responsible for setting the uri of the `ERC1155`.

## Approach
I have audited the target contract in several aspects.
1. `delegateMulti` function converts uint256 array to address, is there any problem with this?
2. Are the permissions for setUri set well?
3. Can a malicious user attack it?
4. Is it compatible with ENSToken?

The results of each audit are shown below.
1. A value of uint256 is passed to `_mintBatch`, `_burnBatch`, which may contain dirty bytes. Since this issue does not cause any loss of funds, I submitted it to QA.
2. `onlyOwner` modifier is present, and Owner is set to deployer.
3. No malicious input has been identified that could be exploited.
4. Compatible with ENSToken and general ERC20Votes.

## Centralization Risks
In `ERC20MultiDelegate`, the admin function is only `setUri`, so the owner cannot do anything malicious on-chain.
And also there is no upgrade logic.

## Architecture Recommendations
As mentioned in the QA report, I would like to make two suggestions on `delegateMulti`.
1. take an address array as an argument.
There is no reason to receive `sources`, `targets` as an array of uint256. it's gassing up the type casting. Use assembly to convert to uint array to save gas.
2. calling `_reimburse`, `createProxyDelegatorAndTransfer` when source, target are null addresses.
The current implementation handles the source or target that is left over at the end of the transfer. Contract can only handle one of the source or target, so in certain scenarios we need to send two transactions.





### Time spent:
12 hours