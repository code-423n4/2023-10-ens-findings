ERC20MultiDelegate is a minimalistic proxy delegation primer, which I found well enough written. Main shortcoming is the stated general application purpose, while for a variety of voting enabled ERC20 out there such minimal implementation doesn't suffice. I.e. while ERC20Votes inheritance is requested for the voting token and certain logic is expected, the fact that in addition to that this token can have an arbitrary mechanics, being literally any smart contract by itself, is overlooked.

Particularly, two main vectors found are `delegateMulti()` flash voting enabling reentrancy for the tokens with transfer hooks and the incompatibility with any balance modification logic of the voting token, i.e. fee on transfers, rebasing, and so on.

Reentrancy vector is amplified by the fact that ERC1155 burning and minting is quite elegantly used for accounting. But such minting have the hook on its own, and the lack of reentracy protection can provide quite a loophole in that setup.

Balance modification vector basically means that in order to support such tokens a Vault like functionality has to be written, which seems excessive for the stated purpose, but is basically necessary as 1-1 correspondence with basic ERC1155 receipts is in fact requested by the logic. I.e. `token` being ERC20Votes doesn't have to be bare ERC20Votes, while ERC20MultiDelegate receipts are almost bare ERC1155, and a representation of such arbitrary `token` with a given straightforward ERC20MultiDelegate implementation can be a big enough ask.

Also, using addresses as ERC1155 ids is an interesting approach, but it needs to be implemented a bit more consistently as now various ids can be minted representing the same address they are *projected* to by truncation (i.e. minting of `1 << 255 + delegate`, `1 << 161 + delegate` is now possible and will represent the same rights for the delegation proxy balance of the delegate) and this can be used for misinforming the users who will deal with those receipts thereafter.

### Time spent:
15 hours