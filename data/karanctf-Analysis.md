The ERC20MultiDelegate contract is designed to allow token holders to delegate voting power to multiple target delegates . It utilizes proxy delegator contracts and ERC1155 for efficient delegation management.

Key Points:\_

**Documentation**: The code would benefit from more comprehensive comments and documentation. While the code is relatively clear, additional comments can enhance readability and make it more accessible to other developers. During audit the documentation wasn't very clear.


**Codebase Quality:** The code is well-organized and follows best practices, with room for improved documentation to enhance readability. But in some places like [1](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L173),[2](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163),[3](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L163),[4](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L198) function naming convention is not followed.
Also during audit found out that a few functions could be merged to avoid confusion.


**Mechanism Flexibility:** The contract's mechanism is flexible and handles various delegation scenarios efficiently. 


### Time spent:
10 hours