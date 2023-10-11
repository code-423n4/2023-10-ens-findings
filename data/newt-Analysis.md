## Analysis report

ENS's ERC20MultiDelegate.sol file is quit solid in my auditing point of view. Though I did find some gas optimization and 3 occurrences of a medium risk finding. Unfortunately with the limited scope it is not impossible but improbable to find a high risk. 

Lines 76 and 81 error messages could be kept under 32 bytes to save gas. If the length is 32 bytes or longer, the slot in which they are defined stores the length of the string * 2 + 1, while their actual data is stored elsewhere. However, if a string is less than 32 bytes, the length * 2 is stored at the least significant byte of it’s storage slot and the actual data of the string is stored starting from the most significant byte in the slot in which it is defined.

Lines 92 and and 95, what is done with the address of the executor once it has been gotten?

Overall the codebase was understandable and straight to the point. Minimal risk has been achieved.

### Time spent:
6 hours