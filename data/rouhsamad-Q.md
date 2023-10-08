# Issue:
The constructor for the contract sets the claimPeriodEnds to the passed _claimPeriodEnds parameter. However, there's no validation to ensure that _claimPeriodEnds is in the future relative to the contract's deployment time.
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ENSToken.sol#L49
# Recommended mitigation steps:
To avoid potential issues, it's advisable to add a validation check within the constructor to ensure that _claimPeriodEnds is greater than the current block.timestamp.




# Issue:
Currently, the smart contract bestows elevated privileges solely to the "owner", which is likely an EOA in this context. While this simplifies access control, it also centralizes authority and exposes the system to potential vulnerabilities, especially if the owner's private key is compromised. High-stake decisions made through this owner-centric model lack the necessary checks and balances, potentially leading to irreversible and damaging actions. (such as sweeping tokens from the contract by a malicious owner)
# Recommended mitigation steps:
role-based access control is a great way to decentralize authority within the contract. It allows assigning specific permissions to different addresses based on their roles.
Example:
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant SWEEPER_ROLE = keccak256("SWEEPER_ROLE");