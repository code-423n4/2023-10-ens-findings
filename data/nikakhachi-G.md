# [G-01] `Math.max(sourcesLength, targetsLength)` can be cached

In the internal function `_delegateMulti`, the following calculation: `Math.max(sourcesLength, targetsLength)` is used twice. First in `require` (line 80) and second time in `for loop` (line 87). It will also get called on every iteration in the loop.

Caching this calculation number in a variable will save gas.

```
File: ERC20MultiDelegate.sol

79:        require(
80:            Math.max(sourcesLength, targetsLength) == amountsLength,
81:            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82:        );
83:
84:        // Iterate until all source and target delegates have been processed.
85:        for (
86:            uint transferIndex = 0;
87:            transferIndex < Math.max(sourcesLength, targetsLength);
88:            transferIndex++
89:        ) {
```

Can be changed to:

```
        uint256 maxSourceOrTargetLength = Math.max(
            sourcesLength,
            targetsLength
        );

        require(
            maxSourceOrTargetLength == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < maxSourceOrTargetLength;
            transferIndex++
        ) {
```

# [G-02] Some equations can be inlined to reduce number of variables declarations

`_delegateMulti`

```
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
Can be changed to:
```
        uint256 targetsLength = targets.length;

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            Math.max(sourcesLength, targetsLength) == amounts.length,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
`_processDelegation`
```
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);
```
Can be changed to:
```
        assert(amount <= getBalanceForDelegate(source));
```
`_reimburse`
```
        address proxyAddressFrom = retrieveProxyContractAddress(token, source);
        token.transferFrom(proxyAddressFrom, msg.sender, amount);
```
Can be changed to:
```
        token.transferFrom(retrieveProxyContractAddress(token, source, msg.sender, amount);
```
`createProxyDelegatorAndTransfer`
```
        address proxyAddress = deployProxyDelegatorIfNeeded(target);
        token.transferFrom(msg.sender, proxyAddress, amount);
```
Can be changed to:
```
        token.transferFrom(msg.sender, deployProxyDelegatorIfNeeded(target), amount);
```
`transferBetweenDelegators`
```
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
        address proxyAddressTo = retrieveProxyContractAddress(token, to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
```
Can be changed to:
```
        token.transferFrom(retrieveProxyContractAddress(token, from), retrieveProxyContractAddress(token, to), amount);
```
`retrieveProxyContractAddress`
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
        return address(uint160(uint256(hash)));
```
Can be changed to:
```
        return
            address(
                uint160(
                    uint256(
                        keccak256(
                            abi.encodePacked(
                                bytes1(0xff),
                                address(this),
                                uint256(0), // salt
                                keccak256(
                                    abi.encodePacked(
                                        type(ERC20ProxyDelegator).creationCode,
                                        abi.encode(_token, _delegate)
                                    )
                                )
                            )
                        )
                    )
                )
            );
```

# [G-03] Unnecessary `_token` argument for `retrieveProxyContractAddress`

`retrieveProxyContractAddress` is a private function and receives the `_token` argument and returns it's proxy contract address. But everytime it's called, the same global `token` variable is passed to it. So instead, we can remove this argument and make the function read the token directly from the storage.

```
File: ERC20MultiDelegate.sol

198:    function retrieveProxyContractAddress(
199:        ERC20Votes _token,
200:        address _delegate
201:    ) private view returns (address) {
202:        bytes memory bytecode = abi.encodePacked(
203:            type(ERC20ProxyDelegator).creationCode, 
204:            abi.encode(_token, _delegate)
205:        );
```
Can be changed to:
```
    function retrieveProxyContractAddress(
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode,
            abi.encode(token, _delegate)
        );
```
 
# [G-04] More likely to revert operations should be executed first

In the `_delegateMulti` function, the second `require` statement is mathematically much more likely to be reverted so it's a good idea to put it as first `require` statement.

```
File: ERC20MultiDelegate.sol

74:        require(
75:            sourcesLength > 0 || targetsLength > 0,
76:            "Delegate: You should provide at least one source or one target delegate"
77:        );
78:
79:        require(
80:            Math.max(sourcesLength, targetsLength) == amountsLength,
81:            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
82:        );
```
Can be changed to:
```
        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
```