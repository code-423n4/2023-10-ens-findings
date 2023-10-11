| *Issue* | *Description* |
|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [L-01] | Possible re-entrancy in Possible re-entrancy in [_delegateMulti](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65) |
| [L-02] | The whole contract does not work with approvals  |
| [L-03] | Quadratic voting is less secure with [ERC20ProxyDelegator](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) |    
| [L-04] | If a user claims to his **ERC20ProxyDelegator** his tokens will be lost | 
| [L-05] | The wrapper will not work with any token |                                                                                                     
| [N-01] | If a user send any vote tokens to [ERC20ProxyDelegator](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol) they will be lost forever |                                                                       


### [L-01] Possible re-entrancy in [_delegateMulti](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65)
There is a re-entracy in `_delegateMulti` where the attack contract is called by **ERC20MultiDelegate**. I didn't have enough time to manage the exploit, because of it I am putting it as low, as it's not a finished exploit.

<details>
<summary>PoC</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

import {Test} from "forge-std/Test.sol";
import {console} from "forge-std/console.sol";
import {ERC20ProxyDelegator,ERC20MultiDelegate} from "../contracts/ERC20MultiDelegate.sol";
import {MyVotes} from "../contracts/MyVotes.sol";
import {IERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";

contract MultiTest is Test{
    address attackerAddress;
    ERC20MultiDelegate multi;
    ERC20ProxyDelegator delegator;
    MyVotes token;
    Attacker2 attacker2;

    function setUp() public {
        vm.label(attackerAddress,"Attaker");
        token = new MyVotes(address(111));
        multi = new ERC20MultiDelegate(token, "idk mate");
        attacker2 = new Attacker2(token,address(multi));
        attackerAddress = address(attacker2);
        token.mint(attackerAddress,1000e18);
    }

    function test_reEnter() public {
        uint256[] memory sources = new uint[](0);
        uint256[] memory targets = new uint[](1);
        uint256[] memory amounts = new uint[](1);
        
        {
            targets[0] = uint256(uint160(attackerAddress));
            amounts[0] = 1000e18;
        }
        vm.startPrank(attackerAddress);
        token.approve(address(multi), 1e24);
        multi.delegateMulti(sources, targets, amounts);
        vm.stopPrank();
    }
}
contract Attacker2{
    MyVotes token;
    ERC20MultiDelegate multi;
    constructor(MyVotes _token,address _multi) {
        token = MyVotes(_token);
        token.approve(_multi,type(uint256).max);
        multi = ERC20MultiDelegate(_multi);
    }
    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
        console.log("onERC1155Received re-entered");
        return this.onERC1155Received.selector;
    }
    function onERC1155BatchReceived(address, address, uint256[] memory, uint256[] memory, bytes memory) public virtual returns (bytes4) {
        console.log("onERC1155BatchReceived re-entered");
        return this.onERC1155BatchReceived.selector;
    }

    function onERC721Received(address, address, uint256, bytes memory) public virtual returns (bytes4) {
        console.log("onERC721Received re-entered");
        return this.onERC721Received.selector;
    }
}
```

```jsx
Logs:
  onERC1155Received re-entered
```

</details>

### [L-02] The whole contract does not work with approvals
The whole **ERC20MultiDelegate** does not work with approvals. This means that is a user approves another user for his 1155 tokens, the other user will no be able to use them with **ERC20MultiDelegate**.

Example:

1. User1 has 200 tokens with id 333 and 200 tokens with id 111.
2. User2 has 200 tokens with id 333.
3. User2 approves User1 for his 200 tokens with id 333.
4. User1 call `delegateMulti` with params for `_processDelegation` to swap the 200 tokens (id: 333) to 200 (id: 111).

However due to the way `_processDelegation` makes the swap with the inside ERC20Vote tokens, user1 swaps **his** 200 (id: 333) tokens instead of user2's.

```solidity
    function _processDelegation(address source,address target,uint256 amount) internal {
        uint256 balance = getBalanceForDelegate(source);
        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }

    function transferBetweenDelegators(address from,address to,uint256 amount) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```

5. User1 has 400 tokens (id: 111) and 0 (id: 333).
6. User2 still has his 200 (id: 333) tokens.

I would suggest to implement a way for approvals to work with the current contract.

### [L-03] Quadratic voting is less secure with ERC20ProxyDelegator
The power of quadratic voting is that large volumes of vote tokens will be square rooted. However with the help of **ERC20ProxyDelegator** and it's ability to instantly delegate to 100+ addresses sibyl  attacks on quadratic voting systems become more possible and with time more common. 

If attacker votes with 10k tokens with a single address => 100 votes. However he can simply split 10k  tokens between 100 addresses and vote with 100 tokens per address (10 votes) => 1000 votes. Applying 10x to his voting power. 

Because this issues is about the whole structure of the system, I am not able to give a guarantied fix. Will leave it for the devs to decide.

### [L-04] If a user claims to his **ERC20ProxyDelegator** his tokens will be lost 
When users claim from **ENSToken** they specifically need to claim to themselves and not to their own **ProxyDelegator**. This is because [claimTokens](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ENSToken.sol#L59-L70) transfers the tokens directly to the `msg.sender` and when transferred to ProxyDelegator, no ERC1155 will be minted to the user.

Example:
1. User1 claims his ENS tokens to himself, and then uses **MultiDelegate** to spread them, in return receiving ERC1155
2. User2 claims his ENS tokens directly to his **ProxyDelegator**

User2 will not receive any ERC1155 tokens and the vote tokens will be stuck forever in his **ProxyDelegator**. He will be able to vote with them, however if he want to delegate them to someone else, he will not be able to do so.

### [L-05] The wrapper will not work with any token
The sponsor explained that the wrapper - **ERC20MultiDelegate** can be used with any vote token and it's not only for ENS to use it. However there is an issue as some vote tokens **disable transfers**. Usually this is done in order for the DAO to have a better time tracking votes, and avoids many vulnerabilities (multiple votings, vote->transfer->vote, ...).

Because they disable the transfer and **ERC20MultiDelegate** works directly by transferring these vote tokens to another contract (ProxyDelegator) the whole contract will not function for such tokens. This is not a massive issue, and for now I am unaware of how it can be fixed (don't make transfers, yet delegate to multiple users), hence I am putting it as a low.

### [N-01] If a user send any vote tokens to ERC20ProxyDelegator they will be lost forever
If a user sends any VoteTokens to **ERC20ProxyDelegator** they will be counted as votes, since this contract delegates to someone, however they will be stuck there forever. 
