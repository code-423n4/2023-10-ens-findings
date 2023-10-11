## [NC-01] Ensure delegator calls `delegateMulti()` from a compliant ERC1155 contract

## Impact & Recommendation
In the protocol, the delegator will mint/burn ERC1155 ownership tokens with IDs represented by target/source delegate addresses. This means that the delegator MUST call the `delegateMulti()` from a contract implementing the ERC1155 specification along with the `ERC1155TokenReceiver` interface to accept transfers/mints mentioned [here](https://eips.ethereum.org/EIPS/eip-1155#specification). More specifically, it should implement the `onERC1155Received()` and `onERC1155BatchReceived()` functions since minting/burning are esentially specialized transfers. It is also explicitly mentioned that these functions should not be called on an EOA.

If not done so, delegation could revert, or worse case if delegator does not receive ownership tokens when calling from contract that does not follow specifications, could cause locked ERC20Votes tokens within the proxy contract, allowing target delegates to retain votes forever

Hence, consider including a warning to make sure users do not call `delegateMulti()` from a contract not implementing the ERC1155 specification, to ensures they will always receive the ERC1155 ownership tokens as intended. 


## [NC-02] Non-compatibility with rebasing tokens

## Impact & Recommendation
If a positive rebasing voting token is utilized, then the  assertion [here](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131) could fail when trying to retrieve some amounts of tokens existing within the proxy addresses, causing some amounts of potentially locked tokens.

If a negative rebasing voting token is utilized, then delegators could potentially retain some ERC1155 ownership tokens without burning them and use it to reimburse tokens from source delegate using other delegator tokens that delegated to the same delegate address (proxy address will be the same).

Hence, include in specification/documentation of contract to not utilize delegation contracts for non-standard ERC-20 tokens such as rebasing tokens.

## [NC-03] Remove unused imports

## Impact & Recommendation
The contract imports OZ's `Address.sol` and `Math.sol` but none of its functionalities are utilized. Consider removing them from the contract.

```diff
- import {Address} from "@openzeppelin/contracts/utils/Address.sol";

  import "@openzeppelin/contracts/access/Ownable.sol";
  import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
  import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
- import "@openzeppelin/contracts/utils/math/Math.sol";
```

