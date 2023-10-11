QA1. function delegateMulti(): We should have zero address check for each address in ``targets`` and zero amount check for each value in ``amounts``. Otherwise, we will assign zero address as the delegates. 

[https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63C1])https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L57-L63C1_

QA2. The constructor of ERC20MultiDelegate fails to pass arguments to its ancestor Ownable. 

The mitigation is as follows:

```javascript
  constructor(
        ERC20Votes _token,
        string memory _metadata_uri
    ) ERC1155(_metadata_uri) Ownable(msg.sender) {
        token = _token;
    }
```

QA3. The retrieveProxyContractAddress() is a private function used by the ERC20MultiDelegate, which always uses ``token``. Therefore the argument of ``_token`` is not necessary.

```diff
function retrieveProxyContractAddress(
-        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
+       ERC20Votes _token = token;
        bytes memory bytecode = abi.encodePacked(
            type(ERC20ProxyDelegator).creationCode, 
            abi.encode(_token, _delegate)
        );
        bytes32 hash = keccak256(
            abi.encodePacked(
                bytes1(0xff),
                address(this),
                uint256(0), // salt
                keccak256(bytecode)
            )
        );
        return address(uint160(uint256(hash)));
    }
```

QA4. I suggest to split the delegateMulti() function into three functions: transferDelegation, reimburseDelegation; and createDelegation as follows. It not only saves gas (due to elimination of many comparisons), but also improve UX (they can pick which function to call based on each of the three cases).

```javascript

    function transferDelegation(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        require(sourcesLength > 0, "empty array");
        require(sourcesLength == targetsLength && targetsLength == amountsLength, "array lengths not equal.");

        for (uint i; i < sourcesLength; i++)  _processDelegation(address(uint160(sources[i])), address(uint160(targets[i])), amounts[i]);

        _burnBatch(msg.sender, sources, amounts);
        _mintBatch(msg.sender, targets, amounts, "");
    }

    function reimburseDelegation(
        uint256[] calldata sources,
        uint256[] calldata amounts
    ) external {
        uint256 sourcesLength = sources.length;
        uint256 amountsLength = amounts.length;

        require(sourcesLength > 0, "empty array");
        require(sourcesLength ==  amountsLength, "array lengths not equal.");

        for (uint i; i < sourcesLength; i++)  _reimburse(address(uint160(sources[i])), amounts[i]);

        _burnBatch(msg.sender, sources, amounts);
    }

    function createDelegation(
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        require(targetsLength > 0, "empty array");
        require(targetsLength == amountsLength, "array lengths not equal.");

        for (uint i; i < targetsLength; i++)  createProxyDelegatorAndTransfer(address(uint160(targets[i])), amounts[i]);

        _mintBatch(msg.sender, targets, amounts, "");
    }
```

QA5. I suggest to allow ERC20ProxyDelegator() to delegate any ERC20 tokens not just for a fixed one:

```javascript
contract ERC20ProxyDelegator {
    address owner;
    address delegate;
     
    constructor(address _delegate) {
        owner = msg.sender;
        delegate = _delegate;
    }

    function delegateToken(ERC20 _token) public {
        require(msg.sender == owner);

        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);

}
```