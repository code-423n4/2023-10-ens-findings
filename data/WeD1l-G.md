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
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85

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

# Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking thefunction as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a paymentwas provided.

```
    function setUri(string memory uri) external onlyOwner {
        _setURI(uri);
    }
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L151

Recommended code：
```
    function setUri(string memory uri) external onlyOwner payable{
        _setURI(uri);
    }
```

# Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this
Refrences:https://book.getfoundry.sh/reference/forge-std/compute-create-address

```
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
```

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L209