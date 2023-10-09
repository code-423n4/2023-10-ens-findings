# G-01 != should be used instead of < in the for loop.
In the for loop, since 1 is added to each iteration, there is no risk of the condition not being met and the for loop not being terminated. Therefore, `<` can safely be replaced with `!=`
```
file: contracts/ERC20MultiDelegate.sol
87:            transferIndex < Math.max(sourcesLength, targetsLength);
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87C1-L87C68


# G-02 _reimburse() should check the ERC1155 token balance of the user to avoid unnecessary code execution
In `_processDelegation`, this verification is also performed, and this exact check should also be implemented in `_reimburse` as it can save a significant amount of gas, as the transaction would revert at the latest when burning the tokens at the end of the function call.
This is the check in the `_processDelegation` function that can be copied to the beginning of the `_reimburse` function:
```
file: contracts/ERC20MultiDelegate.sol
129:        uint256 balance = getBalanceForDelegate(source);
130:
131:        assert(amount <= balance);
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L129C1-L131C35


# G-03 Variables should be declared outside of the for loop
In the for loop, the variables `sourcesLength`, `targetsLength`, and `amountsLength` are newly declared each time. These variables should be declared before the for loop and only assigned within the loop:
```
file: contracts/ERC20MultiDelegate.sol
62:    uint256 amountsLength;
63:    uint256 targetsLength;
64:    uint256 sourcesLength;
65:    function _delegateMulti(
66:        uint256[] calldata sources,
67:        uint256[] calldata targets,
68:        uint256[] calldata amounts
69:    ) internal {
70:        sourcesLength = sources.length;
71:        targetsLength = targets.length;
72:        amountsLength = amounts.length;

```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L70C1-L72C48