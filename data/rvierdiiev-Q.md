## QA-01. ERC20MultiDelegate will not work for voting tokens that don't allow delegate with 0 balance
## Description
When new proxy for delegate is deployed, then in constructor [`delegate` function is called](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L18). And only after construction, [proxy receives tokens](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L160). This means that once proxy calls `delegate`, then it's balance is likely 0.

In case if there will be voting token with overriden delegate function that doesn't allow delegate when you have 0 balance, then this approach will not work. Then construction of proxy will simply revert. As result someone will need to send 1 wei of tokens to the proxy, before construction, in order to be able to deploy it, which is not convenient.
## Recommendation
First transfer tokens and then deploy proxy. 

## QA-02. ERC20MultiDelegate.createProxyDelegatorAndTransfer doesn't emit event.
## Description
When someone delegates his voting power to delegator, then `DelegationProcessed` event should be emitted.
This is done for case, when user [transfers voting power from one delegator to another](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L136) inside `_processDelegation` function, but it isn't done, [when user deploys new delegator and transfer voting power](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155-L161). Because of that it will be harder to track voting power changes using events.

Also i guess, that in case if user just withdraws from delegator, then even should be emitted as well.
## Recommendation
Emit `DelegationProcessed` event inside `createProxyDelegatorAndTransfer` function.

## QA-03. Voting token that doesn't return bool on transferFrom will not work with ERC20MultiDelegate.
## Description
In order to send voting tokens, ERC20MultiDelegate uses `transferFrom` function from ERC20 standart. This function returns `bool` value.
However, not all tokens implement that correctly, so sometimes their `transferFrom` function returns nothing.

In case such token will be used as `token` inside ERC20MultiDelegate, then all calls will revert and contract will not be able to work.
## Recommendation
Use `SafeERC20` extension from OZ to make transfer calls.

## QA-04. Contract has unused library
## Description
```
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
using Address for address;
```

ERC20MultiDelegate has imported Address and Math library, but they are never used inside the contract. These imports should be removed.
## Recommendation
Remove unused libs imports.