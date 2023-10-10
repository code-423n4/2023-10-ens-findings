## 1. Low. Consider overriding `ERC1155.uri()` function
### Vulnerability Details
In protocol ERC1155 token represents delegated amount of ERC20Votes. But current implementation returns the same metadata for all tokenIds.

By default `ERC1155.uri(uint256 tokenId)` returns the same tokenUri for all tokenIds. Often metadata represents properties of that tokenId:
```solidity
    /**
     * @dev See {IERC1155MetadataURI-uri}.
     *
     * This implementation returns the same URI for *all* token types. It relies
     * on the token type ID substitution mechanism
     * https://eips.ethereum.org/EIPS/eip-1155#metadata[defined in the EIP].
     *
     * Clients calling this function must replace the `\{id\}` substring with the
     * actual token type ID.
     */
    function uri(uint256) public view virtual override returns (string memory) {
        return _uri;
    }
```
### Recommended Mitigation Steps
Return dynamic uri according to tokenId:
```solidity
    function uri(uint256) public view virtual override returns (string memory) {
        return abi.encodePacked(_uri, id.toString(), ".json");
    }
```

## 2. Low. ERC20ProxyDelegator.sol is not compatible with COMP token
### Vulnerability Details
Current implementation approves uint256.max amount to sender.
```solidity
    constructor(ERC20Votes _token, address _delegate) {
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }
```
However COMP token [reverts on approved amount > uint96.max](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol#L86-L98):
```solidity
    function approve(address spender, uint rawAmount) external returns (bool) {
        uint96 amount;
        if (rawAmount == type(uint).max) {
            amount = type(uint96).max;
        } else {
            amount = safe96(rawAmount, "Comp::approve: amount exceeds 96 bits");
        }

        ...
    }

    function safe96(uint n, string memory errorMessage) internal pure returns (uint96) {
        require(n < 2**96, errorMessage);
        return uint96(n);
    }
```

### Recommended Mitigation Steps
Use different contract `COMPProxyDelegator` to handle this case:
```diff
- contract ERC20ProxyDelegator {
+ contract COMPProxyDelegator {
    constructor(ERC20Votes _token, address _delegate) {
-        _token.approve(msg.sender, type(uint256).max);
+        _token.approve(msg.sender, type(uint96).max);
        _token.delegate(_delegate);
    }
}
```
And also use updated contract COMPMultiDelegate to deploy special delegator