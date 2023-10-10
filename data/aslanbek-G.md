# [G-01] function _delegateMulti - avoid excessive if-else and ternary operators by splitting the loop into smaller ones
[ERC20MultiDelegate.sol#L85-L108](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L108)

Instead of a single "swiss-army-knife" loop that handles every input via if-else and ternary operators, it will be better to separate the loop into smaller ones that wont have them.

The expected input is: a amounts, s sources, t targets; a = max(s,t). 
```
        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```
For `transferIndex` ∈ `[0; min(sourcesLength, targetsLength))`, only function `_processDelegation` is called. So we're moving it into a separate loop:
```
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }
```
For `transferIndex` ∈ `[min(sourcesLength, targetsLength), max(sourcesLength, targetsLength))`, we create a separate loop (one for sources and one for targets):
```
        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```

Original version:

```
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

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
Optimized version:
```
        uint256 minSourcesTargetsLength = Math.min(
            sourcesLength,
            targetsLength
        );
        for (
            uint transferIndex = 0;
            transferIndex < minSourcesTargetsLength;
            transferIndex++
        ) {
            address source = address(uint160(sources[transferIndex]));
            address target = address(uint160(targets[transferIndex]));
            uint256 amount = amounts[transferIndex];
            _processDelegation(source, target, amount);
        }

        if (sourcesLength > targetsLength) {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < sourcesLength;
                transferIndex++
            ) {
                address source = address(uint160(sources[transferIndex]));
                uint256 amount = amounts[transferIndex];
                _reimburse(source, amount);
            }
        } else {
            for (
                uint transferIndex = minSourcesTargetsLength;
                transferIndex < targetsLength;
                transferIndex++
            ) {
                address target = address(uint160(targets[transferIndex]));
                uint256 amount = amounts[transferIndex];
                createProxyDelegatorAndTransfer(target, amount);
            }
        }
```

# [G-02] Use address.code.length directly instead of caching extcodesize
[ERC20MultiDelegate.sol#L179-L185](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L179-L185)
```diff
    function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
-       uint bytecodeSize;
-       assembly {
-           bytecodeSize := extcodesize(proxyAddress)
-       }

        // if the proxy contract has not been deployed, deploy it
-       if (bytecodeSize == 0) {
+       if (proxyAddress.code.length == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }
```
# [G-03] retrieveProxyContractAddress does not need `_token` parameter - it can be retrieved from contract's bytecode

With [G-02](https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#g02-state-variables-that-are-never-modified-after-deploymentconstructor-should-be-declared-as-constant-or-immutable) fix from the bot report, it's better to retrieve the `token` from contract's bytecode instead of passing it as a parameter every time the function is invoked

```diff
-    ERC20Votes public token;
+    ERC20Votes public immutable token;
```
 


```diff
    function retrieveProxyContractAddress(
-       ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode,
-           abi.encode(_token, _delegate)
+           abi.encode(token, _delegate)
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
    }
```
Remove `token` parameter from the contract at the following lines: 
```
147: address proxyAddressFrom = retrieveProxyContractAddress(token, source);
```
```
168: address proxyAddressFrom = retrieveProxyContractAddress(token, from);
169: address proxyAddressTo = retrieveProxyContractAddress(token, to);
```
```
176: address proxyAddress = retrieveProxyContractAddress(token, delegate);
```