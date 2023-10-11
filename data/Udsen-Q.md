## 1. USE `require` INPLACE OF `assert` STATEMENT FOR CONDITIONAL CHECKS

The `assert` conditional check is used in the `ERC20MultiDelegate._processDelegation` function as shown below:

        assert(amount <= balance); 

But the `assert` is used to ensure `invariants` are held and these `invariants` should never be broken. Hence in the `ERC20MultiDelegate` contract it is recommended to use `require` statement inplace of `assert` since it is the suitable conditional check statement to check the given condition of `amount <= balance`.

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131

## 2. USE AN INPUT VALIDATION CHECK IN THE `ERC20MultiDelegate.setUri` FOR THE PASSED IN INPUT STRING VALUE TO ENSURE IT IS NOT EMPTY

In the `ERC20MultiDelegate.setUri` function there is no input validation check for the passed in `string memory uri` value as an input. Hence it is recommneded to add an input validation check in the `setUri` function to ensure valid input string is passed in to set the `ERC1155` uri. `setUri` function can be modified as shown below to ensure proper input validation is performed.


    function setUri(string memory uri) external onlyOwner {
        require(bytes(uri).length > 0, "Input string is empty");
        _setURI(uri);
    }

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L151-L153

## 3. REDUNDANT EXECUTION OF `Math.max(sourcesLength, targetsLength)` CAN BE REPLACED WITH `amountsLength` VALUE IN THE `ERC20MultiDelegate._delegateMulti` FUNCTION TO OPTIMIZE THE CODE

In the `ERC20MultiDelegate._delegateMulti` function there is a `require` statement to ensure `Math.max(sourcesLength, targetsLength) == amountsLength` as shown below:

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

Hence if the transaction executes beyond the above `require` statement check, it is verified that the `Math.max(sourcesLength, targetsLength) == amountsLength`. 

In the `for` loop which follows the above `require` statement has the following conditional check `transferIndex < Math.max(sourcesLength, targetsLength)` as shown below:

        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {

But there is no requirement to use the `Math.max(sourcesLength, targetsLength)` value and `amountsLength` can be used in its place by replacing the `Math.max(sourcesLength, targetsLength)` which will optimize the code by removing the redundant code execution.

Hence the above `for` loop can be updated as follows:

        for (
            uint transferIndex = 0;
            transferIndex < amountsLength;
            transferIndex++
        ) {

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L79-L82
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85-L89

## 4. IN THE `ERC20ProxyDelegator.constructor`, THERE IS NO INPUT VALIDATION CHECK FOR `_delegate` FOR `address(0)`

In the `ERC20ProxyDelegator.constructor` function there is no input validation check for `_delegate` address. It is recommended to perform an `address(0)` check on the _delegate to ensure valid address is passed into the initialization of the `ERC20ProxyDelegator` contract.

Hence it is recommended to update the `ERC20ProxyDelegator.constructor` function to check for `_delegate` address validity as shown below:


    constructor(ERC20Votes _token, address _delegate) {
        require(_delegate != address(0), "_delegate can not be address(0)");
        _token.approve(msg.sender, type(uint256).max);
        _token.delegate(_delegate);
    }

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L16-L19