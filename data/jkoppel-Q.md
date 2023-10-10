If a fee-on-transfer token is used, then earlier withdrawers effectively get to steal token from later withdrawers.

E.g.:

1. Alice and Bob each delegate 5000 PAX to someone. 9998 PAX arrives in the ProxyDelegator contract for that person. Each person should own 4999 PAX in the contract.
2. Alice and Bob each get 5000 token minted indicating their claim
3. Alice reimburses herself for the full 5000 token. Now only 4998 PAX remains in the ProxyDelegator. Alice has effectively stolen 1 PAX from Bob.

 _delegateMulti should check how much token has actually arrived  in the ProxyDelegator before minting token.