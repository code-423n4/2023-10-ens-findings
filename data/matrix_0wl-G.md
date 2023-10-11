## Gas Optimizations

|        | Issue                                                                                                                   |
| ------ | :---------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                             |
| GAS-2  | STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE                            |
| GAS-3  | IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED |
| GAS-4  | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS                                                          |
| GAS-5  | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE                                        |
| GAS-6  | USE A MORE RECENT VERSION OF SOLIDITY                                                                                   |
| GAS-7  | USE `REQUIRE` INSTEAD OF `ASSERT`                                                                                       |
| GAS-8  | STATE VARIABLES ONLY SET IN THE CONSTRUCTOR SHOULD BE DECLARED IMMUTABLE                                                |
| GAS-9  | USE != 0 INSTEAD OF > 0 FOR UNSIGNED INTEGER COMPARISON                                                                 |
| GAS-10 | USE BYTES32 INSTEAD OF STRING                                                                                           |
| GAS-11 | USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS                                         |
| GAS-12 | USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT                                         |

### [GAS-1] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

148:         token.transferFrom(proxyAddressFrom, msg.sender, amount);

160:         token.transferFrom(msg.sender, proxyAddress, amount);

170:         token.transferFrom(proxyAddressFrom, proxyAddressTo, amount);

```

### [GAS-2] STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE

#### Description:

In several places within the contract, state variables such as token and other variables are accessed multiple times within functions without caching them in local variables. For example, token is accessed multiple times within functions like `_reimburse`, `_processDelegation`, and others without being cached locally. Every time a state variable is accessed, it incurs a gas cost. Caching these variables in local memory variables would minimize gas costs, as it would replace expensive SLOAD operations with cheaper stack reads.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

124: function _processDelegation(
    address source,
    address target,
    uint256 amount,
    ERC20Votes tokenCopy
) internal {
    uint256 balance = getBalanceForDelegate(source, tokenCopy);

    assert(amount <= balance);

    deployProxyDelegatorIfNeeded(target);
    transferBetweenDelegators(source, target, amount);
    emit DelegationProcessed(source, target, amount);
}

144: function _reimburse(
    address source,
    uint256 amount,
    ERC20Votes tokenCopy
) internal {
    address proxyAddressFrom = retrieveProxyContractAddress(tokenCopy, source);
    tokenCopy.transferFrom(proxyAddressFrom, msg.sender, amount);
}

```

### [GAS-3] IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED

#### Description:

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

86:             uint transferIndex = 0;

```

### [GAS-4] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

#### Description:

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

144:     function _reimburse(address source, uint256 amount) internal {

```

### [GAS-5] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

151:     function setUri(string memory uri) external onlyOwner {

```

### [GAS-6] USE A MORE RECENT VERSION OF SOLIDITY

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

2: pragma solidity ^0.8.2;

```

### [GAS-7] USE `REQUIRE` INSTEAD OF `ASSERT`

#### Description:

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

131:         assert(amount <= balance);

```

### [GAS-8] STATE VARIABLES ONLY SET IN THE CONSTRUCTOR SHOULD BE DECLARED IMMUTABLE

#### Description:

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

48:     token = _token;

```

### [GAS-9] USE != 0 INSTEAD OF > 0 FOR UNSIGNED INTEGER COMPARISON

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

75:             sourcesLength > 0 || targetsLength > 0,

75:             sourcesLength > 0 || targetsLength > 0,

110:         if (sourcesLength > 0) {

113:         if (targetsLength > 0) {

```

### [GAS-10] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

46:         string memory _metadata_uri

151:     function setUri(string memory uri) external onlyOwner {

```

### [GAS-11] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

#### Description:

Do not use return at the end of the function.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

173:     function deployProxyDelegatorIfNeeded(
        address delegate
    ) internal returns (address) {
        address proxyAddress = retrieveProxyContractAddress(token, delegate);

        // check if the proxy contract has already been deployed
        uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }

        // if the proxy contract has not been deployed, deploy it
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            emit ProxyDeployed(delegate, proxyAddress);
        }
        return proxyAddress;
    }

192: function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }

198:  function retrieveProxyContractAddress(
        ERC20Votes _token,
        address _delegate
    ) private view returns (address) {
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

### [GAS-12] USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Description:

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/ERC20MultiDelegate.sol

75:             sourcesLength > 0 || targetsLength > 0,

```
