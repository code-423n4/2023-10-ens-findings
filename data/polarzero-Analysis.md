## 1. Overview

The system implements a mechanism to **delegate voting power**, through ERC20Votes compatible tokens, to multiple delegates. Specifically, the `ERC20MultiDelegate` contract allows for **efficiently batching operations**, regardless of their nature:

- *delegating directly* from the user to a chosen delegate;
- transferring a delegation *from one delegate to another*;
- *revoking* a delegation and *withdrawing* the associated tokens.

The system uses the `ERC1155` standard to track the delegated amounts for each delegator, and proxy contracts to actually hold the delegated tokens.

This provides a **secure way to hold these tokens**, and allows for **efficiently tracking** the tokens delegated by individuals, in addition to the native `ERC20Votes` ability to track the voting power of each delegate. All this while ensuring a **direct way for users to revoke any individual delegation**, and **withdraw the associated tokens**, at any given moment.

Simplicity on the surface often stems from quality and efficiency beneath; such is the case with this system.

## 2. Audit approach
Time spent: ~40 hours

### 2.1 First overview

When I started the review, the first thing I did was to get a broader understanding of the system it is supposed to be part of, namely, the Ethereum Name Service. That’s why I started with a first look at the documentation, and mostly some back and forth with an AI, provided with the description of the contest, the code of the file in scope and some additional context. This gave me an abstract understanding of this specific system, how it could be used, as well as its benefits compared to existing similar structures. After a while, I was ready to go deeper into the code.

### 2.2 Environment

The first practical step was to convert the Hardhat tests into a Foundry environment. This would be a good starting point for the following parts, and eventually a good way to understand the system better. That’s when I truly grasped how “simple” it was: a single function that can be called by users, which will crawl through the provided arrays, each time performing one of three possible actions.

### 2.3 Testing

The tests already had 100% coverage, accounting for all the expected cases. My aim was to try to cover edge cases; flood it with large amounts of random arrays, in a progressively more accurate approach. The threat could come from both angles: meticulously crafted input data that would abuse the system, and unexpected or excessively large amounts of data that would break it.

It started with stateless fuzzing, throwing random data to the function. Then gradually less random as I would have it grab random credible inputs, and gradually more organized to perform one of three possible operations: delegate tokens from the user to a delegate, from a delegate to another, and withdraw them from a delegate. Which is the point where I switched to *stateful* fuzzing, and defined invariants.

### 2.4 Threats & invariants

My primary concern in this system was that someone would be able to **steal tokens** from someone else - especially from a proxy contract, or end up with their funds **stuck** in a proxy.

A less concerning, but still significant issue, would be if someone were able to **inflate votes** without actually spending tokens, thus rendering the whole system obsolete.

The invariants mentioned in the details of the contest were the following:
**a.** Tokens should only be transferred between approved delegators.
**b.** The owner should only have the ability to change the URI for ERC1155 metadata.

I supplemented them with the following assumptions:
**c.** The amounts of `ERC20` tokens delegated to an individual should always strictly equal the amount of `ERC1155` tokens owned for the ID corresponding to their address.
**d.** The balance of a proxy contract should always strictly equal the votes of its associated delegate.
**e.** A proxy contract should always contain as many tokens as the cumulative amount its delegate has been trusted with by multiple delegators.

The latter might seem a bit redundant, but it’s never too much.

### 2.5 Reports

The extensive testing and examination of the contract did not reveal - at least to me - any significant bug. Actually, any unexpected behavior at all. At some point, I felt I had exhausted a fair amount of options; I decided to focus on this write-up, and on reporting optimizations and informational “issues”.

## 3. Mechanism review

The `ERC20MultiDelegate` contract deploys/retrieves an individual `ERC20ProxyDelegator` proxy contract for each delegate, which hold the tokens they were delegated by various users.

The actual voting power/weight of each delegate is tracked with the `ERC20Votes` functionalities, saving a series of checkpoints to allow for retrieving their weight at the current time, as well as at a specific date. The amount of tokens distributed across delegates for a user is efficiently tracked by an `ERC1155` token, which associates each delegate’s address (as an ID) to the various balances of their delegators. These balances are “attached” to the amount of the actual token stored in the delegate’s proxy; not as it would *******belong******* to them, but as it is a unique contract *associated* to their address, providing a trust-minimized way to actually store the delegated tokens.

The only operating function accessible to the users is called `delegateMulti` and takes multiple parameters, to perform the various previously mentioned operations. The contract ensures that tokens are transferred correctly and that the `ERC1155` balances reflect these changes.

The following is a simplified call graph of the `ERC20MultiDelegate` contract, highlighting the different steps of the process, especially the three possible outcomes when calling `delegateMulti`. You can refer back to it throughout the analysis.

![Graph.jpg](https://user-images.githubusercontent.com/99199454/274353269-b2d8d80d-8b7a-45a9-9ffe-4b5fbfa79378.jpg)

The following is a **list of key points related to this function**, in the overall context of the contract, and should provide a better understanding of the possibilities. It is followed by a **higher level example of interactions** that can be expected, in an attempt to provide a comprehensive technical and conceptual perception of the system.

### 3.1 Key aspects

**3.1.1 `ERC20MultiDelegate` contract**
**a.** The main function `delegateMulti` allows a user to specify source delegates (from whom tokens are withdrawn), target delegates (to whom tokens are given), and the amounts to be transferred.
**b.** After making sure the highest number of sources or targets is equal to the number of amounts, it goes through each index, opening the following three possible outcomes:
- The index includes **both a source and a target.**
    - Tokens are transferred **from the source's proxy** contract **to the target's proxy** contract.
- The index includes **only a target**
    - Tokens are transferred **from the user** (calling the function) **to the unique proxy** contract associated to the delegate’s address.
- The index includes **only a source**.
    - Tokens are transferred **from the proxy** contract associated to the source, **back to the user** calling the function, thus canceling the delegation and withdrawing the tokens.

**c.** Each of the above described outcomes **will not allow** any of such transfers if the caller is not the actual delegator to the sources passed to the function, or is passing an inflated amount.
**d.** It won’t transfer either if the allowance for the token is insufficient; actually, **it seems to revert** if any of the intended transfers/delegations is **inaccurate**, **abusive**, or simply **not possible**.
**e.** Each **delegate** address can be treated as a **unique token ID** due to the `ERC1155` implementation.
**f.** Proxy contracts for delegates are **created on-demand**. If a delegate doesn't have a proxy contract yet, one is deployed when required. This is done in the `deployProxyDelegatorIfNeeded` function.
**g.** The address of a proxy contract for a specific delegate is **deterministically computed** using the `retrieveProxyContractAddress` function. This ensures that for a given delegate, the proxy contract's address will **always** be the same.

**3.1.2 `ERC20ProxyDelegator` contract**
**h.** Every time this contract is deployed, it approves the `ERC20MultiDelegate` contract to spend all of its ERC20 tokens, so it can transfer them on its behalf.
**i.** Immediately after the approval, the proxy contract delegates its votes to a specified delegate; more precisely, the one whose address was used to compute its own. This means the voting power of the tokens **held by the proxy** will be **assigned to the delegate**.

**3.1.3 Tracking the amount of `ERC1155`**
**j.** Since the ERC1155 contract can manage multiple tokens, each representing a delegate, the balance of a specific delegate/token for an account can be checked using the `balanceOf` function.
**k.** In the context of this system, this `balanceOf` function will return **the exact same amount as `ERC20` tokens delegated** by this account to this specific delegate (ID).
**l.** This is due to the fact that when delegating tokens, the `ERC1155` balances are updated accordingly, making it a record of how much `ERC20` tokens an account has delegated to each delegate.

### 3.2 Step-by-step interactions

**3.2.1 Setting up**
**a.** Alice has a certain amount of `ERC20` tokens in her wallet (inheriting from `ERC20Votes`).
**b.** She identifies multiple proposals or candidates she wishes to support in the upcoming ENS governance vote. She decides she wants to delegate her tokens among them.

**3.2.2 Initial delegation**
**c.** Alice decides to delegate her tokens among three delegates: Bob, Charlie, and Dave.
**d.** She wants to split her tokens equally among them.
**e.** Alice calls the `delegateMulti` function, providing three target addresses - `Bob`, `Charlie`, `Dave` - and the amount for each.
- Internally, the contract:
    - *Checks if proxy contracts already exist for `Bob`, `Charlie`, and `Dave`.*
    - *Deploys proxy contracts for any delegate that doesn't have one.*
    - *Transfers Alice's tokens to these proxy contracts and updates her `ERC1155` balance for each, by minting her the appropriate amount of tokens.*
    - *The proxy contracts then delegate the votes to `Bob`, `Charlie`, and `Dave` respectively.*

**3.2.3 Changing delegation**
**g.** After some time, Alice decides she wants to change her delegation. She wants to remove her delegation from Dave and split their share between Bob and Charlie.
**h.** Alice calls the `delegateMulti` function again. This time, she provides `Dave`'s address as a `source` (indicating she's withdrawing from him) and `Bob` and `Charlie` addresses as `targets`.
- Internally, the contract:
    - *Withdraws the tokens from Dave's proxy contract.*
    - *Splits and redistributes these tokens to Bob's and Charlie’s proxy contracts.*
    - *Updates her `ERC1155` balances for all of them, by burning/minting the associated tokens in batches.*
    - *The proxies then adjust the delegation accordingly.*

**3.2.4 Reclaiming tokens**
**i.** At some point, Alice decides she wants to reclaim some of her tokens from Bob and Charlie.
**j.** She calls the `delegateMulti` function, providing `Bob` and `Charlie`’s addresses as `sources` but doesn't provide any targets.
- Internally, the contract:
    - *Withdraws the tokens from the proxy contracts of both Bob and Charlie.*
    - *Transfers them directly to Alice’s wallet.*
    - *Burns all of her ERC1155 tokens for the IDs corresponding to Bob and Charlie’s addresses.*
    - *The proxy then resets her delegation for both delegates.*

## 4. Architecture recommendations

 The following architectural recommendations can be made.

### **4.1 Batch Processing & gas usage**

The batch processing methods (`_burnBatch`, `_mintBatch`) should be monitored for gas consumption, especially when handling a large array of delegates. The same applies for deploying multiple proxy contracts, when dealing with a wide array of new delegates as targets.

This is especially relevant if an implementation intends to handle gas for users (i.e. meta transactions), which could easily be abused, even by accident.

### **4.2 User interface**

This application requires a thoroughly designed interface. Otherwise, it would be too easy to mess up the array formatting, thereby performing unintended transactions.

While in most cases it would not constitute a significant risk, it *could* in some edge cases. For instance, in the event of a highly disputed vote, any widespread malfunction in the system could have drastic consequences.

## 5. Centralization risks

The only privilege of the owner is the ability to change the `ERC1155` metadata URI (`setUri`), which **does not directly affect the core functionality** of the `ERC20MultiDelegate` contract. Its operations, like delegation, proxy deployment, and transfers, would be left unaffected by any update of this URI.

The only issue would be an inconvenience for the users, due to the **inaccessibility/tempering of information** related to a delegate. Which definitely does **not** constitute a significant centralization risk.

## 6. Systemic risks

### 6.1 Malformed data

The main concern would be poor formatting of the input data for the delegation process, as described in **4.2**.

This would, due to the accurate definition in the contract, in any case, **revert if the data is incorrect**, or do some **reversible non-significant damages**. Meaning, delegating **too much/too few tokens**, **withdrawing them by mistake** or **transferring them from/to an incorrect delegate**; any of this, if easily accessible on a user-friendly interface, would be **easily noticed and fixed without any significant threat**.

### 6.2 Dependencies versions

There are questions concerning the outdated versions of contracts, used in the provided repository, and which versions will be used for deployment.

It is even more unclear whether the versions from the audit package or the latest ones will be utilized, as the contracts referenced in the ENS documentation are deployed using the most recent versions.

This is especially true for packages related to @ensdomains and @openzeppelin, given the huge gap between versions. See in `package.json`:

```json
"dependencies": {
    // 2 years old, latest is 0.0.22 as of 2023-10-11
    "@ensdomains/ens-contracts": "^0.0.7",
    // 2 years old, latest is 5.0.0 as of 2023-10-11
    "@openzeppelin/contracts": "^4.3.1",
    // ...
  },
```

Just a specific example, [the `PublicResolver` contract deployed in tests](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L110) takes two parameters in the older version (the `ENS` and `INameWrapper` contract/interface), whereas the latest one takes four parameters (the same + addresses for a trusted ETH controller and a trusted reverse registrar).

In any case, the audit is intended to cover such cases, regardless if the version is 0.0.7 or above (as it is actually indicated). The point highlighted is the potential benefit of clearly defining the versions of dependencies intended for deployment, as late as possible, and the **possibility of introducing new vulnerabilities in the case of an upgrade *after* the audit is completed**.

## 7. Conclusion

During this audit, which was my first, I first felt a bit disappointed not finding medium/high issues. But, it's actually a testament to the developer's expertise.

Systems implementing new approaches, to address a missing or unusual solution, often thrive when simplified strategically. This specific code might appear complex at first glance, however it's actually rather straightforward, with clever layout beneath the surface.

Depending on the final results, this audit will provide a good reflection of my current skills. Regardless of financial outcomes, I really enjoyed the process; it was great auditing a system I'm truly interested in, while improving my testing skills, with in-depth analysis and formal verification methods.

Looking forward, I’m eager to dive into more implementations from the ENS team and help with the review and security assessment.

### Time spent:
40 hours