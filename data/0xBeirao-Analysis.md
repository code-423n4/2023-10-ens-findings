### General

I've spent 1 day on this audit and covered the unique contract in scope.

When I saw ENS + only 200 Sloc: I was expecting 0 vulnerabilities on this contest.

### Approach

The main external library I focused on was OpenZeppelin's ERC20Vote. I tried to understand this library as deep as possible and quickly realized that the `ERC20ProxyDelegator` implemented its voting delegation perfectly. I then looked at the ENSToken contract to see if there was anything that could go wrong with the proxy delegator.

### Architecture Recommendations

For a contract running on the Ethereum mainnet, the `ERC20ProxyDelegator` contract is not really gas optimized. There are simple and safe gas optimization techniques that can be used here.

The overall architecture is well built and it was a pleasure to review it. As I reported, I think the proxy used can be more resilient to external actions such as airdrops or sending tokens directly to proxies.

### Qualitative Analysis

The code quality is very mature, but there is a lack of documentation for this delegation feature.

The test suite is quite comprehensive.

### Centralization risks

None. The architecture is completely permissionless.

### Systemic Risks

Since the entire platform is not upgradeable, migration in the event of a vulnerability. The team should prepare multiple recovery scenarios and set up appropriate action channels to prepare for such eventualities.

### Time spent:
8 hours