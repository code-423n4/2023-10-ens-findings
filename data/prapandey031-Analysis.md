ENS ERC20MultiDelegate.sol analysis:
---------------------

**Approach taken in evaluating the codebase:**
----------------------
The approach was a manual go-through of all the contracts involved. 
Initially, the inherited OZ contracts were thoroughly understood. After this, the ERC20MultiDelegate.sol was thoroughly understood as well.

**Codebase quality analysis:**
----------------------
The codebase is simple to understand. The use of ERC1155 to set up the mechanism for multiple delegates is efficient.
The codebase is solid to hack.
However, the codebase lacks sufficient NatSpec comments.

**Centralization risks:**
----------------------
The ```setUri(string memory uri)``` function uses ```onlyOwner``` modifier that instills a bit of centralisation risk into the codebase.

**Time Spent:**
----------------------
10 hours

### Time spent:
10 hours