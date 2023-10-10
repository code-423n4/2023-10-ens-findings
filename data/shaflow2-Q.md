
## Potential risks of address brute-forcing

The contract utilizes a fixed salt for checking the ERC20ProxyDelegator, which introduces potential risks of address brute-forcing. A malicious individual could calculate a salt and deploy contracts with malicious functions preemptively at the ERC20ProxyDelegate location. For instance, they could modify the _delegate parameter to their own address in order to acquire votes. While calculating the salt may be challenging, it is not impossible if there is a profit incentive involved.



GitHub[15](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L15)
    src:contracts/ERC20MultiDelegate.sol

    15    contract ERC20ProxyDelegator {
    16        constructor(ERC20Votes _token, address _delegate) {
    17            _token.approve(msg.sender, type(uint256).max);
    18            _token.delegate(_delegate);
    19        }
    20    }

GitHub[173](https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L173C1-L190C6)

    src:contracts/ERC20MultiDelegate.sol

    173    function deployProxyDelegatorIfNeeded(
    174        address delegate
    175    ) internal returns (address) {
    176        address proxyAddress = retrieveProxyContractAddress(token, delegate);
    177
    178        // check if the proxy contract has already been deployed
    179        uint bytecodeSize;
    180        assembly {
    181            bytecodeSize := extcodesize(proxyAddress)
    182        }
    183
    184        // if the proxy contract has not been deployed, deploy it
    185        if (bytecodeSize == 0) {
    186            new ERC20ProxyDelegator{salt: 0}(token, delegate);
    187            emit ProxyDeployed(delegate, proxyAddress);
    188        }
    189        return proxyAddress;
    190    }
