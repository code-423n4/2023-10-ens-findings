# Low

## [L-01] `_safeTransferFrom()` can be used to circumvent blocklists

**Issue Description:**
Using the `_safeTransferFrom()` functionality of `ERC1155`, users can circumvent blocklists and still transfer tokens to another address after being blocklisted. This can happen in the following form:

1. User A delegates all his tokens to User B using the `ERC20MultiDelegate`.
2. User A gets blacklisted, so all transfers from/to his address of the ERC20 token are reverted.
3. User A uses `safeTransferFrom()` to transfer his `ERC1155` tokens to User C.
4. User C is able to withdraw the tokens out of the proxy.

**Recommended Mitigation Steps:**
This problem can be fixed in two ways. The first one being to override the `_safeTransferFrom()` functionality and make it revert on any call, effectively making the `ERC1155` tokens not transferrable. The second way would be to add an optional check to the blocklist of the underlying `ERC20` token before transferring any `ERC1155` tokens.

---
## [L-02] Tokens sent to the contract by accident can get stuck

**Issue Description:**
Some users might, by accident or due to incorrectly understanding the contracts' functionality, send tokens directly to the contract or proxies to delegate them. As there is no functionality to sweep/rescue tokens, these tokens will stay stuck forever.

**Recommended Mitigation Steps:**
Deciding if this issue needs to be fixed is up to the project team. The team could either keep the functionality as it is, which would offer a higher security level in case of ownership corruption but leaves tokens sent incorrectly stuck. If the protocol team decides on wanting a functionality to rescue tokens, a simple function that is only callable by the owner can be added which transfers the full balance of a provided token to the owner.

---
## [L-03] Transfers with 0 amounts are possible
[Line 65](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65)

**Issue Description:**
The current implementation allows for users to call all three different functionalities of `delegateMulti()` with an amount of 0. This does not lead to any direct vulnerabilities but is incorrect behavior as delegating or reimbursing an amount of 0 does not make any sense.

**POC**
A simple testcase that shows the incorrect functionality:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/ENSToken.sol";
import "../src/ERC20MultiDelegate.sol";

contract FuzzingTest is Test {
    ENSToken public token;
    ERC20MultiDelegate public delegate;
    uint256 constant TOTAL_TOKENS = type(uint224).max / 2;
    address user1;
    address user2;
    address user3;

    function setUp() public {
        token = new ENSToken(TOTAL_TOKENS, 0, block.timestamp);
        delegate = new ERC20MultiDelegate(ERC20Votes(token), "");

        user1 = vm.addr(0xdeadbeef);
        user2 = vm.addr(0xbeefdead);
        user3 = vm.addr(0xdeaddead);

        token.transfer(user1, TOTAL_TOKENS);
    }

    function testZeroAmounts() public {
        vm.startPrank(user1);

        //Approve so that the delegate can transfer the tokens
        token.approve(address(delegate), type(uint256).max);

        //Delegation of zero
        uint256[] memory sources1 = new uint256[](0);
        uint256[] memory targets1 = new uint256[](1);
        uint256[] memory amounts = new uint256[](1);
        targets1[0] = uint256(uint160(user2));
        amounts[0] = 0;
        
        delegate.delegateMulti(sources1, targets1, amounts);

        //Transfer of zero
        uint256[] memory sources2 = new uint256[](1);
        uint256[] memory targets2 = new uint256[](1);
        sources2[0] = uint256(uint160(user2));
        targets2[0] = uint256(uint160(user3));

        delegate.delegateMulti(sources2, targets2, amounts);

        //Reimbursement of zero
        uint256[] memory sources3 = new uint256[](1);
        uint256[] memory targets3 = new uint256[](0);
        sources3[0] = uint256(uint160(user3));

        delegate.delegateMulti(sources3, targets3, amounts);
    }
}
```

**Recommended Mitigation Steps:**
Check for 0 values passed in amount and either skip the loop iteration in that case, saving gas or revert.

---
## [L-04] `source` can be the same address as `target`
[Line 124](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L124)

**Issue Description:**
When transferring between two proxies, the user can provide the same address as the target as well as the source. This is not intended behavior and will lead to confusing events being emitted.

**POC**
A simple testcase that shows the incorrect functionality:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/ENSToken.sol";
import "../src/ERC20MultiDelegate.sol";

contract FuzzingTest is Test {
    ENSToken public token;
    ERC20MultiDelegate public delegate;
    uint256 constant TOTAL_TOKENS = type(uint224).max / 2;
    address user1;
    address user2;
    address user3;

    function setUp() public {
        token = new ENSToken(TOTAL_TOKENS, 0, block.timestamp);
        delegate = new ERC20MultiDelegate(ERC20Votes(token), "");

        user1 = vm.addr(0xdeadbeef);
        user2 = vm.addr(0xbeefdead);
        user3 = vm.addr(0xdeaddead);

        token.transfer(user1, TOTAL_TOKENS);
    }

    function testTransferToSameAddress() public {
        vm.startPrank(user1);

        //Approve so that the delegate can transfer the tokens
        token.approve(address(delegate), type(uint256).max);

        uint256[] memory sources1 = new uint256[](0);
        uint256[] memory targets1 = new uint256[](1);
        uint256[] memory amounts1 = new uint256[](1);
        targets1[0] = uint256(uint160(user2));
        amounts1[0] = TOTAL_TOKENS;

        delegate.delegateMulti(sources1, targets1, amounts1);

        //User delegates from user2 -> user2
        delegate.delegateMulti(targets1, targets1, amounts1);
    }
}
```

**Recommended Mitigation Steps:**
To fix this issue, it is recommended to add an additional requirement that reverts in case of both addresses being the same in the `_processDelegation()` function. This could look like this:

```solidity
require(source != target, "Transfer to the same address is not intended");
```
---
# Non-Critical (NC) Findings
## [NC-01]  New delegation and reimbursement are missing events
[Line 144](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144)
[Line 155](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L155)

**Issue Description:**
The contract includes the event `DelegationProcessed` which is emitted when delegated votes get transferred from one delegate to another. The issue is that this is only one of the 3 cases that the contract handles. In the case of newly delegating votes and also in the case of retrieving earlier delegated votes no event is emitted.

**Recommended Mitigation Steps:**
I would recommend either using the event `DelegationProcessed` in these cases and leave the source/target as 0, like it is done with `transferFrom()` for burning/minting in some token implementations. If an extra event for each case makes more sense to the developers, I would recommend adding 2 new events and emitting them at the end of the function calls.

---
## [NC-02]  Incorrect comment in `_reimburse()`
[Line 144](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144)

**Issue Description:**
The comments in the `_reimburse()` function state the functionality as "Reimburses any remaining source amounts back to the delegator after the delegation transfer process." and "Transfer the remaining source amount or the full source amount (if no remaining amount) to the delegator," which is incorrect. This function can be used to reimburse an arbitrary user-chosen amount from a proxy back to himself.

**Recommended Mitigation Steps:**
Change the definition to "Reimburses a user-provided amount (that is less than the `ERC1155` balance the user has for the delegate) back to the user." and remove the comment inside the function.

---
## [NC-03] Missing indexing in event
[Line 36](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L36)

**Issue Description:**
The event `DelegationProcessed` includes three parameters that could all be indexed. In the current implementation, only two of those are indexed.

**Recommended Mitigation Steps:**
Also index the third parameter `amount`:

```solidity
event DelegationProcessed(
	address indexed from,
	address indexed to,
	uint256 indexed amount
);
```

---
## [NC-04] Grammatical error in testcase naming
[Tests Line 138](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L138)
[Tests Line 288](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L288)
[Tests Line 345](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L345)
[Tests Line 688](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L688)

**Issue Description:**
There are multiple test cases that are named grammatically incorrectly:

1. 'should be able to delegate multiple delegates in behalf of user'
2. 'should be able to re-delegate multiple delegates in behalf of user (1:1)'
3. 'should be able to re-delegate multiple delegates in behalf of user (many:many)'
4. 'should revert if allowance is lesser than provided amount' 

**Recommended Mitigation Steps:**
Rename the test cases to:

1. 'should be able to delegate multiple delegates on behalf of user'
2. 'should be able to re-delegate multiple delegates on behalf of user (1:1)'
3. 'should be able to re-delegate multiple delegates on behalf of user (many:many)'
4. 'should revert if allowance is less than provided amount'
---
## [NC-05] Grammatical error in contract descriptions
[Line 23](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L23)

**Issue Description:**
At the start of the `ERC20MultiDelegate` contract, a comment is placed that should describe the utility of the contract. This comment states "@dev A utility contract to let delegators to pick multiple delegate". This is grammatically incorrect.

**Recommended Mitigation Steps:**
Adapt the comment to "@dev A utility contract that lets delegators pick multiple delegates."
