# Issue:
The constructor for the contract sets the claimPeriodEnds to the passed _claimPeriodEnds parameter. However, there's no validation to ensure that _claimPeriodEnds is in the future relative to the contract's deployment time.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ENSToken.sol#L49

# Recommended mitigation steps:
To avoid potential issues, it's advisable to add a validation check within the constructor to ensure that _claimPeriodEnds is greater than the current block.timestamp.
