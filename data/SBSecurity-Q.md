### [Low Risk](notion://www.notion.so/Dopex-Code4rena-d041b7f9146846c78801ff8cc7f30cd3?p=6c3270398be04527a2cdf446b9bbd785&pm=s#low-risk)

| Count | Title |
| --- | --- |
| [L-01] | No check if source == target in _processDelegation |
| [L-02] | No check if amount == 0 |
| [L-03] | Tokens and Ether mistakenly sent to contracts cannot be recovered |
| [L-04] | Everyone can delegate to themselves |
| [L-05] | Delegator can pass an address of a proxy which has already delegated to him |

### [Non-Critical](notion://www.notion.so/Dopex-Code4rena-d041b7f9146846c78801ff8cc7f30cd3?p=6c3270398be04527a2cdf446b9bbd785&pm=s#non-critical)

| Count | Title |
| --- | --- |
| [NC-01] | Missing event for important changes |
| [N-02] | Unnecessary library usage |

| Total Non-Critical Issues | 2 |
| --- | --- |

## Low Risk

## [L-01] No check if source == target in `_processDelegation`

## Impact

If we provide the same source and target addresses, the function will execute successfully but won't result in any modifications.
This is because it invokes the **`token.transferFrom()`** function, which merely deducts the specified amount from the source address and then immediately credits it back to the same address. This behavior occurs because the default implementation of **`ERC20:transferFrom`** does not validate whether the source and target addresses are the same.

Maybe add a check if `source == target` to prevent the user from paying for gas.

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L163-L171

```diff
163	function transferBetweenDelegators(
164	  address from,
165	  address to,
166	  uint256 amount
167	) internal {
+         require(from != to)
169	  address proxyAddressFrom = retrieveProxyContractAddress(token, from);
170	  address proxyAddressTo = retrieveProxyContractAddress(token, to);
171	  token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
172	}
```

## [L-02] No check if `amount == 0`

## Impact

There is no validation whether amount transferred is greater than 0. As a result many proxies will be deployed but they won’t have any funds and therefore will result in a waste of gas and not giving any value to the delegator.

## [L-03] **Tokens and Ether mistakenly sent to contracts cannot be recovered**

## Impact

The `ERC20MultiDelegate` and `ERC20ProxyDelegator` contracts lack functions for `sweep/recover` mistakenly sent tokens or Ether.

Users might assume they should transfer their ENS tokens to the `ERC20MultiDelegate`, expecting it to forward them to the proxy.

Alternatively, they may see any proxy address from the mempool and send their tokens there, under the assumption that this action delegates their votes.

Consider adding this type of function and restrict it to only be callable by the protocol admin.

## [L-04] Everyone can delegate to themselves

The **`_delegateMulti()`** function allows self-delegation because it doesn't check if the target is equal to **`msg.sender`**.The **`ERC20Votes:delegate`** function also lacks this check, resulting in the ability to self-delegate which goes against the contract's intended purpose.

Consider adding a check if `msg.sender == target`.

## [L-05] Delegator can pass an address of a proxy which has already delegated to him

Anyone can establish a "chain delegation" by passing a proxy linked to their own address as the target. In doing so, they delegate to the proxy address (target), which in turn creates another proxy, ultimately leading to the delegation being routed back to the original delegator.

1. Alice delegates 100 tokens to Bob (deploy Proxy1)
2. Carl delegates 50 tokens to Alice (deploy Proxy2)
3. Alice changes her delegation from Bob to Proxy2 (deploy Proxy3), that way she transfers the Vote tokens to Proxy3 which delegates to Proxy2 which delegates to Alice.

## Non-Critical

## **[N‑01]** Missing event for important changes

No events are emitted when delegate from user to proxy and from proxy to user.

In `_reimburse()`:

```diff
144 function _reimburse(address source, uint256 amount) internal {
145     // Transfer the remaining source amount or the full source amount
146     // (if no remaining amount) to the delegator
147     address proxyAddressFrom = retrieveProxyContractAddress(token, source);
148     token.transferFrom(proxyAddressFrom, msg.sender, amount);
+       emit DelegationProcessed(proxyAddressFrom, msg.sender, amount);
149 }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L144-L149

In `createProxyDelegatorAndTransfer()`:

```diff
155 function createProxyDelegatorAndTransfer(
156     address target,
157     uint256 amount
158 ) internal {
159     address proxyAddress = deployProxyDelegatorIfNeeded(target);
160     token.transferFrom(msg.sender, proxyAddress, amount);
+       emit DelegationProcessed(msg.sender, proxyAddress, amount);
161 }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L155-L161

## [N-02] Unnecessary library usage

`ERC20MultiDelegate` contract uses the OZ `Address` library for the address type, but has never used any of its features.

These lines can safely be removed from the contract:

- Line 4 `import {Address} from "@openzeppelin/contracts/utils/Address.sol";`
- Line 26 `using Address for address;`