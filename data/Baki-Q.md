# L-01 Address library of openzeppelin is redundant.
We are importing OZs Address.sol , but it is not used anywhere in the code. So it can be removed.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L4

# L-02 Use a newer version of solidity.
Currently, we are using `0.8.2`, which is fairly old. We recommend using a newer version like 0.8.17 or above without floating pragma.
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L2

