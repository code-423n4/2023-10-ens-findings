# [L-01] - Sending tokens to proxies will delegate the power to vote, but it will never be able to be recovered.

## Impact

If a user sends ENS tokens directly to an `ERC20ProxyDelegator` address, the tokens will be delegated, but the user will not be able to recover their tokens.

This is because each ERC1155 token is paired with the number of ENS tokens stored in the delegation proxy. Directly sending tokens will result in an unrecoverable remnant.

## Proof of Concept

Someone send ENS token to an `ERC20ProxyDelegator` address to delegate his voting power.

When he wants to withdraw his token, he realizes that directly sent tokens are lost.

## Recommended Mitigation Steps

I think a good way to handle this is to make each `ERC20ProxyDelegator` an ERC4626 vault.

This way, when the `ERC20ProxyDelegator` balance increases, the user will be able to claim the due portion of the accrued balance from the directly sent tokens.

Then `shares` from the ERC4626 standard should be owned by the `ERC20MultiDelegate` contract and paired (for access control) with the ERC1155 corresponding to the deployed proxy. For that, only `ERC20MultiDelegate` should be able to call `deposit`, `mint`, `withdraw` and `redeem`.

```solidity
contract ERC20ProxyDelegator is ERC4626, Ownable {

  constructor(ERC20Votes _token, address _delegate) {
      _token.approve(msg.sender, type(uint256).max);
      _token.delegate(_delegate);
  }

  function deposit(
      uint256 assets,
      address receiver
  ) public override onlyOwner returns (uint256 shares) {
      super.deposit(assets, receiver);
  }

  function mint(
      uint256 shares,
      address receiver
  ) public override onlyOwner returns (uint256 assets) {
      super.mint(shares, receiver);
  }

  function withdraw(
      uint256 assets,
      address receiver,
      address owner
  ) public override onlyOwner returns (uint256 shares) {
      super.withdraw(assets, receiver, owner);
  }

  function redeem(
      uint256 shares,
      address receiver,
      address owner
  ) public virtual onlyOwner returns (uint256 assets) {
      super.redeem(shares, receiver, owner);
  }
}
```