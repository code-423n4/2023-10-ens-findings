# 1. Using max() function inside the for loop

## Description

At the following location in the code, there is a function max from OZ Math.sol inside the for loop, it is a waste of gas.

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
```

https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L87C10-L87C10

## Remediation

I would recommend writing a new variable before a for loop and then using it inside the loop in favor of saving gas.

```
uint transferIndex;
```

# 2. Default value initialization

## Description

At the following location in the code, there is a variable initialized to its default value. This is already the default behavior of Solidity and it becomes therefore redundant and a waste of gas.

```
uint transferIndex = 0;
```

https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L86C36-L86C36

## Remediation

I would recommend removing the initialization to a default value in favor of saving gas.

```
uint transferIndex;
```