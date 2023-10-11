## QA-01. ERC20MultiDelegate will not work for voting tokens that don't allow delegate with 0 balance
## Description
When new proxy for delegate is deployed, then in constructor [`delegate` function is called](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L18). And only after construction, [proxy receives tokens](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L160). This means that once proxy calls `delegate`, then it's balance is likely 0.

In case if there will be voting token with overriden delegate function that doesn't allow delegate when you have 0 balance, then this approach will not work. Then construction of proxy will simply revert. As result someone will need to send 1 wei of tokens to the proxy, before construction, in order to be able to deploy it, which is not convenient.
## Recommendation
First transfer tokens and then deploy proxy. 