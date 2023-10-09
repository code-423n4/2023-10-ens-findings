# Variables: No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address...). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

## Issue 1

        for (
            uint transferIndex = 0;

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L86

        for (
            uint transferIndex = 0;

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85C1-L86C36

# Issue 2

Remove empty string(error message) to save gas

        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L113-L114

Recommendation: _mintBatch(msg.sender, targets, amounts[:targetsLength]);


