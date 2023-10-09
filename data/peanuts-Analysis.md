### 1. Approach taken in evaluating the codebase

- Looked through the test files to get the overall idea of the contract
- Read ERC20Votes contract, Votes contract and ERC1155 to better understand the protocol
- Recreated the test suite with the help of wardens in the channel
- Tested end to end scenarios and check if there's any issue with the delegation process using foundry

### 2. Summary and Explanation of the protocol
- The protocol only has one public callable function, which is `delegateMulti()`. The delegation process will start on this function.
- The prerequisite is that the user that calls `delegateMulti()` must have some tokens to delegate. It is possible to have zero tokens as well just to create the proxy
- There are three parameters to fill in `delegateMulti()`: sources, targets, amounts

```
    function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
```
- The first time this function is called by someone, the sources array is usually left empty, like so: []. The targets refer to the people that the user intends to delegate to. The amount refer to how much the user wants to delegate. For example, if user A wants to delegate 10e18 each to Alice and Bob, he will call `delegateMulti([],[Alice,Bob], [10e18,10e18])`
- For every target, user A will create a proxy contract (if not created yet) and transfer the funds to the proxy contract.
```
    function createProxyDelegatorAndTransfer(
        address target,
        uint256 amount
    ) internal {
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
    }
   function deployProxyDelegatorIfNeeded(
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
```
- The proxy contract will then approve the use of the tokens from ERC20MultiDelegate and call delegate.
```
contract ERC20ProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
```
- When everything is finished, user A will not have the 20 tokens. The 20 tokens will be deposited equally into Alice's proxy and Bob's proxy.
- An ERC1155 token is minted for every target to user A. This is to keep track of where user A sent the funds to.

From here, user A can do a few things.
1. User A can retrieve his delegated tokens by calling `delegateMulti()`. This time, the source will be [Alice,Bob] and the target will be empty [].
- Once the tokens are withdrawn from the proxy contract, Alice and Bob no longer has any tokens. The ERC1155 that user A has to track the balance of his funds will be adjusted accordingly
- It is noted that user A can partially retrieve his delegated tokens. For example, user A can withdraw 5 tokens each from Alice and Bob's proxy contract. He can then withdraw another 5 tokens at another time

2. User A can delegate Alice and Bob tokens to two other users. The source will be [Alice,Bob], the target will be [Charlie, Derrick], and the amounts will be whatever Alice and Bob has.
- Note that Alice and Bob can partially or fully delegate their funds to the two new targets. 
- Note that the deployer can withdraw partially delegated funds. Once delegated to another person, the deployer can withdraw funds from Charlie and Derrick directly.

### 3. Architecture recommendations

Scenarios tested
- single to single delegation
- single to multi delegation 
- multi to multi delegation
- multi to single delegation

1. There is just one slight inconvenience about the delegation process. The sources cannot combine their amount and split it to the targets 
- Deployer gives 20e18 to Alice and Bob -> source = [], target = [Alice,Bob], amount = [10e18,10e18]
- Alice and Bob cannot redelegate to 3 people properly -> source = [Alice,Bob], target = [Charlie,Derrick,Ernest], amount = [15e18,2.5e18,2.5e18] will not work
- This is because one proxy will only point to another proxy. For example, Alice only has a maximum of 10e18 tokens to delegate. She cannot delegate 15e18 tokens to Charlie, although the combined amount of Alice is Bob is enough to delegate to three people

This can be rectified by calling the source to the target first, then delegating the tokens again
- Deployer gives 20e18 to Alice and Bob -> source = [], target = [Alice,Bob], amount = [10e18,10e18]
- Alice and Bob gives back 20e18 to the deployer -> source = [Alice,Bob], target = [], amount = [10e18,10e18]
- Deployer gives 20e18 to Charlie,Derrick,Ernest -> source = [], target = [Charlie,Derrick,Ernest], amount = [15e18,2.5e18,2.5e18]

2. Tokens deposited directly to the proxy contract cannot be withdrawn
- This can happen on accident or if people don't know that the real delegator is the deployer instead of the proxy, and sends funds to the proxy contract instead. Probably good to set a emergencyWithdraw function in the proxy, to which only the deployer or creator of the proxy can access.

3. Proxy contract approving type(uint256).max may not work on all ERC20 tokens
- Some tokens such as USDT needs to approve 0 first. Some tokens only have a max approval of uint96. The protocol must be careful if the contract uses tokens other than ENS

### 4. Centralization risks
- Not much centralization risk in this contract, except for setting the uri which does not impact the funds in any way

### 5. Mechanism review
- Seems like a good way of extending the ERC20Votes function to allow for multi delegates.

### Time spent:
15 hours