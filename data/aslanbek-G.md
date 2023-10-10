# [G-01] function _delegateMulti - save gas on if-else and ternary operators by splitting the loop into smaller ones
[ERC20MultiDelegate.sol#L85-L108](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L108)

Instead of the "swiss-army-knife" loop that handles every input via if-else and ternary operators, it will be better to separate the loop into smaller ones that wont have them.

The expected input is: a amounts, s sources, t targets; a = max(s,t). 

```
        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );
```

1. First loop - for indexes in `[0 : min(s,t))`. Ternary operator is removed: Sources and Targets are guaranteed to be provided; Only `_processDelegation` will be invoked.
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

2. The second loop is for `[min(s,t) : max(s,t))`.

a) If s > t, we use a loop with `_reimburse`. Only Sources are needed and they are guaranteed to be provided.

b) If s <= t, we use a loop with `createProxyDelegatorAndTransfer`. Only Targets are needed and they are guaranteed to be provided.

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



[The test](https://gist.github.com/aslanbekaibimov/da4f4c8a0454f8a41277773e92a71c94) creates 5 delegatees, then moves their voting power to 5 other delegatees.

`solc = 0.8.21` (Sponsor confirmed that they will be using the latest version)
`runs = 200`
```
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2143100                                                      | 11491           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 882638          | 949097 | 949097 | 1015556 | 2       |

| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2171929                                                      | 11635           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 881537          | 947828 | 947828 | 1014120 | 2       |

| Savings                                                      | 1101            | 1269   | 1269   | 1436    |         |
```
Deployment cost will increase by ~29k gas, which, as you can see, will pay off very quickly.


# [G-02] retrieveProxyContractAddress does not need `_token` parameter - it can be retrieved from the contract's bytecode

With the [G-02](https://github.com/code-423n4/2023-10-ens/blob/main/bot-report.md#g02-state-variables-that-are-never-modified-after-deploymentconstructor-should-be-declared-as-constant-or-immutable) optimization from the bot report, it's better to retrieve the `token` from contract's bytecode inside the function's body, instead of passing it as a parameter every time the function is invoked.

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
Remove `token` parameter from the following lines: 
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

```
|--------------------------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2143100                                                      | 11491           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 882638          | 949097 | 949097 | 1015556 | 2       |

|--------------------------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2122226                                                      | 11366           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 882518          | 949017 | 949017 | 1015516 | 2       | 


| Savings                                                      | 120             | 80     | 80     | 40      |         | 

Deployment savings - 20874 gas
```

# [G-03] Redundant retrieveProxyContractAddress

```diff
    function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

 -      deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
```
    function transferBetweenDelegators(
        address from,
        address to,
        uint256 amount
    ) internal {
        address proxyAddressFrom = retrieveProxyContractAddress(token, from);
-       address proxyAddressTo = retrieveProxyContractAddress(token, to);
+       address proxyAddressTo = deployProxyDelegatorIfNeeded(to);
        token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);
    }
```
```
| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2125456                                                      | 11245           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 885663          | 952517 | 952517 | 1019371 | 2       |

| Deployment Cost                                              | Deployment Size |        |        |         |         |
| 2120056                                                      | 11218           |        |        |         |         |
| Function Name                                                | min             | avg    | median | max     | # calls |
| delegateMulti                                                | 873450          | 946410 | 946410 | 1019371 | 2       |

| Savings                                                      | 12213           | 6107   | 6107   | 0       |         |

Deployment savings - 5400 gas
```