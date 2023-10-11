## 1. Codebase Analysis

The `ERC20MultiDelegate` enables token/vote transfers to targets or facilitates reimbursements, where different source/target combinations are permissible, provided the total amounts match up.  The contract deploys a proxy delegate for each of the targets, that receives tokens, then grants max allowance to the ERC20MultiDelegate contract before delegating votes to the specified target address. For tracking ownership, it inherits from ERC1155, where `tokenId` in the function `balanceOf(sender, tokenId)` , is the address of the target converted to a uint256.


## 2. Possible Use Cases
The flow chart in this [post](https://x.com/__alexxander_/status/1711340589478732042?s=20) presents a very good summary of protocol flow and mechanism. The following table summarizes the potential use cases of the protocol, assuming the following checks are met:

- `sourcesLength` and/or `targetsLength` matches this [check](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L74-L77) where delegator should provide at least one source or one target delegate

<br/>

- `amountsLength` matches this [check](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79-L82), that is the number of amounts must be equal to the greater of number of source/target delegate. This prevents unintended delegation/transfer/burning/minting of tokens.

<br/>

- `_burnBatch()`: [This function call](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L111) when `sourcesLength > 0` is esentially the "access control" here. The delegator must own the ERC1155 ownership token represented by source address (represented by a uint256), to be allowed to perform delegation from source delegate to target delegate.

Note that ERC1155 tokens are transferrable and owner of this tokens can approve other users to spend this tokens via `ERC1155.setApprovalForAll()`. This means delegation/reimbursement/Transfers can be performed from other addresses.

| Scenario | Actions | Use Case | 
|:--:|:--:|:--:|
| `sourcesLength = 0, targetsLength > 0` | `createProxyDelegatorAndTransfer()` | Transfer ERC20Votes tokens from delegator himself to the target delegate proxy (deploy a new proxy if not already deployed) |
| `sourcesLength > 0, targetsLength = 0` | `_reimburse()` | Reimburse ERC20Votes tokens from source delegate proxy contract back to delegator himself|
| `sourcesLength > 0, targetsLength > 0, sourcesLength == targetsLength `| `_processDelegation()` | Transfer ERC20Votes tokens from source delegate to target delegate |
| `sourcesLength > 0, targetLengths > 0, sourcesLength > targetLength `| `_processDelegation()`, `_reimburse()` | Transfer ERC20Votes tokens from source delegate to target delegate, Perform reimbursement to delegator for last source delegate |
| `sourcesLength > 0, targetLengths > 0, sourcesLength < targetLength `| `_processDelegation()`, `createProxyDelegatorAndTransfer()` | Transfer ERC20Votes tokens from source delegate to target delegate, Perform transfer of tokens from delegator himself to last target delegate |

### 3. Improvements/Risks

### 3.1 Add a unique chainId identifier
To further improve the usability of the contract (possibly across multiple EVM compatible chains), a unique chainId could be included when deploying a new `ERC20ProxyDelegator` contract. 

### 3.2 Delegation does not work as intended if number of source delegates is greater than number of target delegates or vice versa
As mentioned in one of my submissions, the contract does not allow delegating from a single source delegate to multiple different targets and vice versa, causing unexpected results.

1. Delegate from 2 source delegates to a single target delegate, it results in the user having to redelegate to the single target delegate after getting reimbursed. It also means the second source delegate may potentially fewer votes for that duration of time before redelegation.

<br/>

2. Delegate from a single source delegate to 2 target delegate., it results in delegation being processed from the user instead of the singular source delegate. This results in the user having to reimburse from the intended source delegate. It also means source delegate would have a larger amount of votes before reimbursement is performed by delegator.


## 4. Centralization Risks
The only centralization risk involves allowing the owner to change the ERC1155 URI at any point of time, which describes the meta data and information about each token present in the contract.



### Time spent:
36 hours