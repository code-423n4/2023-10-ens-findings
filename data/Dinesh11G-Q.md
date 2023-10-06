## Title

Multiple Minor Issues in ERC20MultiDelegate Smart Contract

## Impact

The identified issues are relatively minor in nature and do not pose significant security threats. However, they can affect the contract's functionality, readability, and gas efficiency.

## Proof of Concept

1. **Access Control**: The contract does not have comprehensive access control for various functions, allowing anyone to execute certain operations. [Code](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L75)

   ```solidity
   require(
       sourcesLength > 0 || targetsLength > 0,
       "Delegate: You should provide at least one source or one target delegate"
   );
   require(
       Math.max(sourcesLength, targetsLength) == amountsLength,
       "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
   );
   ```

## Tools Used

Manual code review and analysis were performed.

## Recommended Mitigation Steps

1. **Access Control**: Implement more granular access control mechanisms for functions, ensuring that only authorized users or roles can perform certain operations. Consider using OpenZeppelin's Access Control Contracts.

2. **Error Messages**: Enhance error messages to provide clear and informative feedback to users, helping them understand why a transaction might fail.