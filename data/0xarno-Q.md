## [L-01] Make `ERC20Votes public token` immutable in `ERC20MultiDelegate` contract!
```diff
- ERC20Votes public token;
+ ERC20Votes public immutable token;
```
### Lines of code
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L28