QA1. function delegateMulti(): We should have zero address check for each address in ``targets`` and zero amount check for each value in ``amounts``. Otherwise, we will assign zero address as the delegates. 

[https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63C1])https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63C1_

QA2. The constructor of ERC20MultiDelegate fails to pass arguments to its ancestor Ownable. 

The mitigation is as follows:

```javascript
  constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) Ownable(msg.sender) {
        token = _token;
    }
```