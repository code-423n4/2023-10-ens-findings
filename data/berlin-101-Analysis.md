## Approach
- Familiarized myself with some basics by reading docs of ENS to have some context knowledge (https://docs.ens.domains/)
- Did a first manual walkthrough of the the ERC20MultiDelegate.sol and ENSToken.sol
- Left some @audit markers in code (QA and issue ideas)
- Followed the issue ideas and wrote POCs where an issue could be confirmed
- Collected all QA findings and submitted the QA report
- Wrote the Analysis as a last step

## Mechanism Design
- It was not apparent at first why the ERC1155 tokens were minted with the id (address to uint) of the vote delegates. But after a while it became clear that it is "just" for keeping track of delegates tokens in an easy way.
- The overall mechanism does not support some edge cases of ERC20 tokens that lie outside the realm of "malicious", "fee on transfer" or "rebasing" tokens. The team explained during the audit that the ERC20MultiDelegate.sol contract should be audited with "any" ERC20 token being wrapped by "ERC20Votes" excluding the aformenetioned cases. ERC20 tokens implementing a block/blacklist and those not supporting a maximum approval of uint256 are unsupported. They do not fall into the categories listed by the protocol.

## Observations and obstacles during the review
- The lack of documentation was decreasing the speed of the security review a bit. How ERC20MultiDelegate.sol is used I had to reverse engineer from the tests.

## Systemic / Centralization risks
- No larger concerns or systemic or centralization risk were identified. There is only one function thas has an onlyOwner modifier.

## Code quality
| Category       | Feedback                                                                                                             |
| -------------- | -------------------------------------------------------------------------------------------------------------------- |
| Documentation  | Documentation was missing.
| Code Comments  | Code comments overall were sparse. I had to go through the tests to understand the contracts better. Some more comments in code would have helped. |
| Code Syntax    | Code Syntax was good. I guess some automated formatting was applied. The botrace discovered some inconsistencies.    |
| Organization   | The codebase very small and well-organized. No issue here.                                                            |
| Unit Testing   | The codebase had a fully functional Hardhat suite of tests. Instructions were provided how to run tests. Fuzz-testing could not be found. |

## Insights and learnings
- The pattern of using a Math.max or a Math.min in order to check multiple values in one statement was discovered the first time and considered interesting. E.g.
```Solidity
require(
	Math.max(sourcesLength, targetsLength) == amountsLength,
	"Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
	);
```
- The pattern of minting and burning of ERC1155 tokens to keep overview of how many tokens have been delegated to which delegate (using their address converted to uint) and using a proxy in between to not delegate tokens to delegates directly was interesting as well.

## Time spent: 20 hours

### Time spent:
0 hours