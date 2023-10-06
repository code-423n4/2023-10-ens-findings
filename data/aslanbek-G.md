# [G-01] Call balanceOf internally 
[ERC20MultiDelegate.sol#L195](https://github.com/code-423n4/2023-10-ens/blob/1adbe2cce191140657b8bccffab85103953bdccb/contracts/ERC20MultiDelegate.sol#L195) 

```diff
    function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
-       return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
+       return balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```