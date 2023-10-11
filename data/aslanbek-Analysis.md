# Mechanism review

The current ERC20Votes implementation allows a ERC20Votes token holder to delegate their **whole** voting power either to another user or to themselves. If they want to split their voting power between more than one delegatee, they need to transfer some of their tokens to a different account and delegate from there, and repeat that for every delegatee, spending a ton of gas along the way.

ERC20MultiDelegate solves this problem: it allows token holders to cheaply and permissionessly split their voting power between delegatees.

Token holders are able to:

1. Lock their ERC20Votes token and distribute their voting power in any arbitrary proportion between however much delegatees they want, in one call.

2. Move an arbitrary amount of their voting power between delegatees.

3. Reimburse locked tokens back to themselves.

4. Transfer ERC-1155's to EOAs or `ERC1155TokenReceiver`s. These tokens represent voting power delegated to a delegatee, and can be burned in return for ERC20Votes tokens locked in the respective ERC20ProxyDelegator contract.




# Centralization Risks

The contract is almost fully decentralized: the only thing that owner can change is URI, which is arguably irrelevant for the contract's purpose.

# Systemic risks

Users can transfer their ERC1155 tokens, redeemable for tokens locked in a ERC20ProxyDelegator. While a user has to deliberately transfer tokens to another account for this to happen, it formally breaks the ["Tokens should only be transferred between approved delegators"](https://github.com/code-423n4/2023-10-ens#:~:text=Tokens%20should%20only%20be%20transferred%20between%20approved%20delegators.) invariant.

If this is not intended, consider overriding ERC-1155 `_safeTransferFrom` and `_safeBatchTransferFrom` to always revert.

# Architecture recommendations

To have a nice feature of ERC-1155 tokenIds matching delegatees' addresses, there has to be either upcasting user inputs from `address` to `uint256`, or downcasting from `uint256` to `address`. 

The developers chose the latter, but it ended up creating a minor issue described in L-01, where users can create tokens with id >= 2^160. 

The issue can be solved :

1. By using SafeCast for not allowing inputs >= 2^160.

- Cheaper gas-wise.

2. By changing user inputs from `uint256[]` to `address[]`, and overriding ERC-1155 `_mintBatch` and `_burnBatch` so that they upcast `address` to `uint256`. 

- Slightly better UX, because delegators can just copy-paste delegatees' addresses.

# Approach taken in evaluating the codebase

It took me some time to get familiar with OpenZeppelin's Governor, ERC20Votes and ERC-1155.

After that, I started looking for vulnerabilities and optimizations in small (2-4 hours) sessions, and managed to find some.

# Codebase quality analysis

The codebase is well-written. No major vulnerabilities or optimizations were detected. While only a brief README was provided, the Sponsor was very responsive and covered every question.







### Time spent:
25 hours