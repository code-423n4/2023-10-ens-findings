# Quality Assurance Report

| QA     | Issues                                            | Instances |
|--------|---------------------------------------------------|-----------|
| [N-01] | Missing event emission for critical state changes | 4         |
| [N-02] | Unnecessary typecast to uint256                   | 1         |

### Number of Non-Critical issues: 5 instances across 2 issues
### Number of Low-severity issues: 0
### Total number of issues: 5 instances across 2 issues

## [N-01] Missing event emission for critical state changes

There are 4 instances of this issue:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L15C1-L21C1

Missing event emission when new delegator instance is created.
```solidity
File: ERC20MultiDelegate.sol
16:     constructor(ERC20Votes _token, address _delegate) {
17:         _token.approve(msg.sender, type(uint256).max);
18:         _token.delegate(_delegate);
19:         //@audit NC - missing event emission when new delegator instance is created
20:     }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L44C1-L48C7

Missing event emission when `token` and `uri` are set. Note: ERC1155 does not emit an event when setting a uri as seen [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/793d92a3331538d126033cbacb1ee5b8a7d95adc/contracts/token/ERC1155/ERC1155.sol#L282C1-L284C6).
```solidity
File: ERC20MultiDelegate.sol
45:     constructor(
46:         ERC20Votes _token,
47:         string memory _metadata_uri
48:     ) ERC1155(_metadata_uri) {
49:         token = _token;
50:         //@audit NC - missing event emission for both token and uri state settings
51:     }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6

Missing event emission when internal function finishes execution.
```solidity
File: ERC20MultiDelegate.sol
60:     function delegateMulti(
61:         uint256[] calldata sources,
62:         uint256[] calldata targets,
63:         uint256[] calldata amounts
64:     ) external {
65:         _delegateMulti(sources, targets, amounts);
66:     }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151C1-L153C6

```solidity
File: ERC20MultiDelegate.sol
160:     function setUri(string memory uri) external onlyOwner {
161:         _setURI(uri);
162:         //@audit NC - missing event emission
163:     }
```

## [N-02] Unnecessary typecast to uint256

There is 1 instance of this issue:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L210

```solidity
File: ERC20MultiDelegate.sol
220:                 uint256(0), // salt //@audit NC - unnecessary typecast
```

