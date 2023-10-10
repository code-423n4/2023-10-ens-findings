## [L-1] wrong implementation on require statement as stated in the comment 
    [L-79](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79)
  ```
 require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
Recommendation : fix typo error 
 ```
require(
            amountsLength => Math.max(sourcesLength, targetsLength) ,
            "Delegate: The number of amounts must be equal to or  greater than the number of sources or targets"
        );
```
## [L-2] check for `` sourcesLength == targetsLength``
if both are same amount then `` Math.max and Math.min ``will also be the same so , else if statement won't be used and `` _reimburse , createProxyDelegatorAndTransfer `` is never called inside the loop but it can be called outside the loop manually by the user.

```

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```

 Recommendation : add check
 ```
require(sourcesLength != targetsLength);
```
## [L-3] `` _mintbatch and _burnbatch `` doesnt follow ERC 115.
  contract ERC20MultiDelegate uses openzeppelin ERC 115.
from https://docs.openzeppelin.com/contracts/3.x/api/token/erc1155#ERC1155-_mint-address-uint256-uint256-bytes- 
>> Requirements:
ids and amounts must have the same length.

which the function doesn't follows it 
``` 
    require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
           );
```
here `` Math.max``  Returns the largest of two numbers. so , there is a single variable equal to ``amountslength ``

```
        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }
```
as stated both should be equal but here only one is equal which might cause ``errors during mint and burn`` .
Recommendation :
``` 
if (sourcesLength > 0 && sourceslength == amountslength) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0 && targetsLength == amountslength) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }
```
## [L-4] missing  `` require(amount <= balance) ``check for _reimburse function 
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L144C14-L144C24
```
  function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }

```
Recommendation:
```
  function _reimburse(address source, uint256 amount) internal {
        // Transfer the remaining source amount or the full source amount
        // (if no remaining amount) to the delegator
 uint256 balance = getBalanceForDelegate(source);
        require(amount <= balance);
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
    }
```
## [N-1]  use require statement instead of assert
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131C35-L131C35
```
require(amount <= balance); 
```
## [N-2] keep keccak256 inside the memory
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L202
```
   bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode, 
            abi.encode(_token, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
```
Recommendation:
```
   bytes memory bytecode = keccak256(abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode, 
            abi.encode(_token, _delegate))
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                bytecode
            )
        );
```