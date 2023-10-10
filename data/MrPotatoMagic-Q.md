# Quality Assurance Report

| QA     | Issues                                                                               | Instances |
|--------|--------------------------------------------------------------------------------------|-----------|
| [N-01] | Missing event emission for critical state changes                                    | 4         |
| [N-02] | Unnecessary typecast to uint256                                                      | 1         |
| [N-03] | Remove unnecessary code                                                              | 3         |
| [L-01] | Missing getUri() function that returns baseUri appended with a user specific tokenId | 1         |

### Number of Non-Critical issues: 8 instances across 3 issues
### Number of Low-severity issues: 1 instance across 1 issue
### Total number of issues: 9 instances across 4 issues

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

## [N-03] Remove unnecessary code

There are 3 instances of this issue:

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6

Although following the pattern of external functions calling internal functions is recommended, the [delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6) function just calls internal [_delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65) function with the passed parameters. This is an unnecessary step. Additionally, there are no complex interactions when calling from the external to internal function, thus making it safe to remove.

Remove this external [delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57C1-L63C6) function below and make the internal function [_delegateMulti()](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L65) to external.
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

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L86C32-L86C35

Since unsigned integer variables are 0 by default, there is no need to initialize them to 0.

```solidity
86: uint transferIndex = 0;
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L198

Remove the parameter ERC20Votes _token from function `retrieveProxyContractAddress`. The function `retrieveProxyContractAddress` is called by functions `_reimburse`, `transferBetweenDelegators` and `deployProxyDelegatorIfNeeded`. These functions always pass the state variable `token` as parameter to the `retrieveProxyContractAddress`. This creates extra gas that can be avoided by just using the state variable `token` in the `retrieveProxyContractAddress` function directly.

Instead of this:
```solidity
File: ERC20MultiDelegate.sol
232:     function retrieveProxyContractAddress(
233:         ERC20Votes _token,
234:         address _delegate
235:     ) private view returns (address) {
236:         bytes memory bytecode = abi.encodePacked(
237:             type(ERC20ProxyDelegator).creationCode, 
238:             abi.encode(_token, _delegate)
239:         );
240:         bytes32 hash = keccak256(
241:             abi.encodePacked(
242:                 bytes1(0xff),
243:                 address(this),               
244:                 uint256(0), // salt
245:                 keccak256(bytecode)
246:             )
247:         );
248:         return address(uint160(uint256(hash)));
249:     }
```
Use this: **(Note: Make sure to make changes in functions `_reimburse`, `transferBetweenDelegators` and `deployProxyDelegatorIfNeeded` when passing parameters to `retrieveProxyContractAddress`)**
```solidity
File: ERC20MultiDelegate.sol
232:     function retrieveProxyContractAddress(
233:         address _delegate //@audit Now only one parameter
234:     ) private view returns (address) {
235:         bytes memory bytecode = abi.encodePacked(
236:             type(ERC20ProxyDelegator).creationCode, 
237:             abi.encode(token, _delegate) //@audit directly using state variable "token"
238:         );
239:         bytes32 hash = keccak256(
240:             abi.encodePacked(
241:                 bytes1(0xff),
242:                 address(this),               
243:                 uint256(0), // salt
244:                 keccak256(bytecode)
245:             )
246:         );
247:         return address(uint160(uint256(hash)));
248:     }
```

## [L-01] Missing getUri() function that returns baseUri appended with a user specific tokenId

Implementing a token type ID substitution mechanism ([as mentioned by OpenZeppelin here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/793d92a3331538d126033cbacb1ee5b8a7d95adc/contracts/token/ERC1155/ERC1155.sol#L47C1-L59C6)) is a widespread practice in contracts that use the ERC1155 standard. 

This is because instead of the user having to manually append their tokenId themselves, the getUri() function provides it directly when the user provides their tokenId. Additionally, a getUri() function can be helpful to read uri data for a specific tokenId directly from the state since it ensures that the data is upto date and removes the frontend from the equation to handle the appending of the tokenId. It also serves as an onchain pointer for a specific tokenId that is some metadata stored offchain in the bucket of a IPFS solution.

Small discussion between Sponsor and I which confirms that a data uri will be used to display metadata on the user frontend:

![](https://user-images.githubusercontent.com/109625274/274043221-b3808025-c315-487b-86df-652313b90dfe.png)

Solution:
```aolidity
    function getUri(uint256 tokenId) external view returns(string memory) {
        string memory tokenID = Strings.toString(tokenId);
        string memory baseUri =  uri(tokenID); //passing tokenID here is not necessary since it just returns base uri
        return string(abi.encodePacked(baseUri, tokenID));
    }
```