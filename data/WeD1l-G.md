# Use ```++transferIndex``` instead of ```transferIndex++``` that less gas cost.
```
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )
```
After optimization：
```
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            ++transferIndex
        )
```

# Use constants instead of type(uintx).max and type(uintx).min
using type(uintx).max and type(uintx).min can result in higher gas costs because it involves a runtime operation to calculate the maximum value at runtime.This calculation is performed every time the expression is evaluated.
To save gas, it is recommended to use constants instead of type(uintx).max to represent the maximum value. By declaring a constant with the maximum value, the value is known at compile-time and does not require any runtime calculations.
  
```
file: contracts/ERC20MultiDelegate.sol
17          _token.approve(msg.sender, type(uint256).max);

80            Math.max(sourcesLength, targetsLength) == amountsLength,

87          transferIndex < Math.max(sourcesLength, targetsLength);

98            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L17
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L80
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98




https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85
