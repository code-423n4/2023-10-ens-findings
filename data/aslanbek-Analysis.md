# Mechanism review

The current ERC20Votes implementation allows a ERC20Votes token holder to delegate their **whole** voting power either to another user or to themselves. If they want to split their voting power between more than one delegates, they need to transfer some of their tokens to a different account and delegate from there, and repeat that for every delegate, spending a ton of gas along the way.

ERC20MultiDelegate solves this problem: it allows token holders to cheaply and permissionessly split their voting power between delegates.

Token holders are able to:

1. Lock their ERC20Votes token and distribute their voting power in any arbitrary proportion between however many delegates they want, in one call.

![FirstDelegation.png](https://user-images.githubusercontent.com/85546446/274298514-fdd64e2e-b134-4277-96f1-390a5c6e183e.png)

2. Move an arbitrary amount of voting power between delegates.

![AdjustingVotingPower.png](https://user-images.githubusercontent.com/85546446/274232660-49951268-f897-4522-a739-386488a79947.png)

3. Reimburse locked tokens back to themselves.

![Reimburse.png](https://user-images.githubusercontent.com/85546446/274232768-bc27ffe8-8d98-4fd1-826f-d9ae0ab96dfc.png)

4. Transfer ERC-1155's to EOAs or `ERC1155TokenReceiver`s. These tokens represent voting power delegated to a delegate, and can be burned in return for ERC20Votes tokens locked in the respective ERC20ProxyDelegator contract.

# Centralization Risks

The contract is almost fully decentralized: the only thing that the owner can change is URI, which is arguably irrelevant for the contract's purpose.

# Architecture recommendations

To have a very useful feature of ERC-1155 tokenIds matching delegates' addresses, there has to be either upcasting of user inputs from `address` to `uint256`, or downcasting from `uint256` to `address`. 

The developers chose the latter, but it ended up creating a minor issue described in L-01, where users can create tokens with id >= 2^160. 

The issue can be solved:

1. By using SafeCast for not allowing inputs >= 2^160.

- Cheaper gas-wise.

2. By changing user inputs from `uint256[]` to `address[]`, and overriding ERC-1155 `_mintBatch` and `_burnBatch` so that they upcast `address` to `uint256`. 

- Slightly better UX, because delegators can just copy-paste delegates' addresses.

# Approach taken in evaluating the codebase

It took me some time to get familiar with OpenZeppelin's Governor, ERC20Votes and ERC-1155.

After that, I started looking for vulnerabilities and optimizations in small (2-4 hours) sessions, and managed to find some minor ones.

# Codebase quality analysis

The codebase is written with respect to the best security practices. No major vulnerabilities or optimizations were detected. 

While only a brief README was provided, the Sponsor was very responsive and covered every question.


### Time spent:
25 hours