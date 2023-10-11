(low) No 0 address check:
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L44C2-L49C6

The line has no checks for the 0 address for the token field:

```
    constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) {
        token = _token;
    }
```