## [N-01] Unclear comments can induce confusion on the type of input parameters

The following is not an issue, but rather an information about some misleading comment, [in ERC20MultiDelegate at L41](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L41).

The contract does not take an address, but rather an instance of `ERC20Votes` as an input.

```solidity
/**
     * @dev Constructor.
     * @param _token The ERC20 token address
     * @param _metadata_uri ERC1155 metadata uri
     */
    constructor(
        ERC20Votes _token, // This is actually not the token address
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) {
        token = _token;
    }
```

## [N-02] Upgrades on dependencies to avoid using outdated packages

### Description

The contract uses old packages as dependencies (2 years old).

These could be updated, to inherit from much more recent and considered safer libraries. Especially as these address more recent security concerns (e.g. being explicit for the address that is passed to `Ownable`). The following is taken from the Analysis report, to highlight the potential issues it could induce, especially when upgrading the system.

> Just a specific example, [the `PublicResolver` contract deployed in tests](https://github.com/code-423n4/2023-10-ens/blob/main/test/delegatemulti.js#L110) takes two parameters in the older version (the `ENS` and `INameWrapper` contract/interface), whereas the latest one takes four parameters (the same + addresses for a trusted ETH controller and a trusted reverse registrar).
> 

> […] The point highlighted is the potential benefit of clearly defining the versions of dependencies intended for deployment, as late as possible, and the **possibility of introducing new vulnerabilities in the case of an upgrade *after* the audit is completed**.
> 

```json
"dependencies": {
    // 2 years old, latest is 0.0.22 as of 2023-10-11
    "@ensdomains/ens-contracts": "^0.0.7",
    // 2 years old, latest is 5.0.0 as of 2023-10-11
    "@openzeppelin/contracts": "^4.3.1",
    // ...
  },
```

### Recommendations

Update the contracts imported from OpenZeppelin [to the latest version (v5.0.0)](https://www.npmjs.com/package/@openzeppelin/contracts/v/5.0.0).

```solidity
// ERC20MultiDelegate.sol

// All the following will be updated
import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {Context} from "@openzeppelin/contracts/utils/Context.sol";
import {ERC1155} from "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import {ERC20Votes} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";
```

Add the explicit statement in the `constructor`:

```diff
- constructor(ERC20Votes _token, string memory _metadata_uri) ERC1155(_metadata_uri) {
+ constructor(ERC20Votes _token, string memory _metadata_uri) ERC1155(_metadata_uri) Ownable(msg.sender) {
  token = _token;
}
```

This will require using a version of Solidity `≥ 0.8.20`.