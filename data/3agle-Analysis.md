# Analysis for ENS Protocol
## Index
- [Codebase Quality](#codebase-quality)
- [Mechanism Review](#mechanism-review)
- [User Flows](#user-flows)
- [Architecture Recommendation](#architecture-recommendation)
- [Key Takeaways](#key-takeaways)

---

## Codebase quality

### **Strengths**

1. Small Codebase
2. Previous Audits

### **Weaknesses**

1. Lack of Documentation

## Mechanism Review

- During the audit, several key principles of the protocol were identified:
    1. **Delegating Voting Power**: Users have the capability to delegate their voting power to multiple other users within a single transaction.
    2. **Proxy Management**: The `ERC20Votes` token are held by proxy contracts (managed by `ERC20MultiDelegate`), and these proxies delegate their voting power to users, establishing a decentralized governance system.
    3. **Balances Tracking**: To efficiently track delegated balances of users, the protocol employs `ERC1155` tokens, providing a robust mechanism for managing and representing these balances.
- Following is the architecture of the codebase:

![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_ee1dwq.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_ee1dwq.png)

## User Flows

### 1. **Deposit**
- In this user flow, a user delegates their tokens to another user.
    
![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_1_e7zysq.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_1_e7zysq.png)
    
### 2. **Withdraw**
- In this user flow, a user withdraws their delegation from another user.
    
![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_2_wnnhmw.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_2_wnnhmw.png)
    
### 3. **Transfer**
- In this user flow, a user transfers their delegation power from User A to User B.
    
![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_3_j1yyo7.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696926023/ENS/Untitled_3_j1yyo7.png)
    
## Architecture Recommendation

- In the way things currently work, users have the flexibility to do various actions all in one go, bundled together within a single transaction.

```solidity
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
       
// (Additional code removed for brevity)

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                createProxyDelegatorAndTransfer(target, amount);
            }

// (Additional code removed for brevity)  
  
    }
```

- However, it's crucial to understand that this approach can sometimes run into issues, like running out of gas during the transaction. This usually happens because handling all these different financial tasks in one go can be pretty resource-intensive. Plus, these tasks often involve interactions with multiple users and their accounts.
- To make things smoother and more efficient while also keeping gas costs in check, it's recommended to reorganize the code by separating each financial task into its own function. Below, we're outlining what these individual functions might look like:

```solidity
 	function _transfer(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        require(
            sources.length == targets.length && targets.length == amounts.length
        );

        for (uint index = 0; index < sources.length; index++) {
            _processDelegation(sources[i], targets[i], amount[i]);
        }

        _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
    }

    function _deposit(
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        require(targets.length == amounts.length);

        for (uint index = 0; index < targets.length; index++) {
            createProxyDelegatorAndTransfer(targets[i], amount[i]);
        }
        _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
    }

    function _withdraw(
        uint256[] calldata sources,
        uint256[] calldata amounts
    ) internal {
        require(sources.length == amounts.length);

        for (uint index = 0; index < sources.length; index++) {
            _reimburse(sources[i], amount[i]);
        }
        _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
    }
```

## Key Takeaways

- Using ERC1155 to manage accounting by creating and destroying tokens is quite a unique approach in itself.
- It deviates from the more common methods, adding an element of innovation and distinctiveness to the accounting process.
- In conclusion, to enhance the codebase quality, it is important to consider the architectural recommendations to optimize gas usage and maintain a high level of efficiency.

### Time spent:
35 hours