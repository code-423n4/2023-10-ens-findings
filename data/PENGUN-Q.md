## The values passed as id for _burnBatch and _mintBatch are uint256, not address.
`sources` and `targets` are uint256 arrays, and in actual use, each element is converted to an address.
In the last part of `delegateMulti`, `_burnBatch` and `_mintBatch` use the `sources` and `targets` arrays as they are, so the values with dirty bytes are applied.
This means that when you delegate to address `0x1234...1234`, the minted id could be `0x7777...1234...1234`.
This can cause problems with front-end integration and confuse users when ERC1155 is tradable.

To solve this, take the `delegateMulti` argument as an addresse array.

## No check for null address target when `_processDelegation`
there is no check for null address so vote will delegate to null address when user mistake.

## Restrictions on the behavior of delegateMulti
Here's how delegateMulti works
if (transferIndex < Math.min(sourcesLength, targetsLength)):
	1. Get the balance of sources. `ERC1155(this).balanceOf(msg.sender, source)
	2. balance must be greater than amount.
	3. If there is no ProxyContract for target, deploy it.
        4. transferFrom(source, target, amount)
	
else if (transferIndex < sourcesLength):
	1. call reimburse(source, amount)
	2. Derive the proxy address for source
	3. transfer token proxy -> msg.sender

else if (transferIndex < targetsLength)
	1. call createProxyDelegatorAndTransfer(target, amount)
	2. Derive a proxy address for target
	3. transfer token msg.sender -> proxy 

Due to the structure of the if statement, the following scenarios cannot be operated in a single transaction.

1. User A delegate 100 to User B
2. User A want add 100 token and change delegater to User C
3. User A's first call delegateMulti([B],[],[100])
3. User A's second call delegateMulti([],[C],[200])

To fix this, call `_reimburse`, `createProxyDelegatorAndTransfer` by checking if the source or target is a null address.
