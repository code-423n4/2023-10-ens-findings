| *Issue* | *Description*                                                                                                                                              |
|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [L-01]  | Possible re-entrancy in Possible re-entrancy in [_delegateMulti](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65) |
| [L-02]  | The whole contract does not work with approvals                                                                                                            |
| [N-01]  | If a user send any vote tokens to ERC20ProxyDelegator they will be lost forever                                                                            |


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

### [N-01] If a user send any vote tokens to ERC20ProxyDelegator they will be lost forever
If a user sends any VoteTokens to **ERC20ProxyDelegator** they will be counted as votes, since this contract delegates to someone, however they will be stuck there forever. 