## QA
---

### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol

```solidity
// place external functions before all the others
151:    function setUri(string memory uri) external onlyOwner {
```

---

### natSpec missing [2]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol

```solidity
16:    constructor(ERC20Votes _token, address _delegate) {
32:    event ProxyDeployed(address indexed delegate, address proxyAddress);
33:    event DelegationProcessed(
65:    function _delegateMulti(
151:    function setUri(string memory uri) external onlyOwner {
155:    function createProxyDelegatorAndTransfer(
163:    function transferBetweenDelegators(
173:    function deployProxyDelegatorIfNeeded(
192:    function getBalanceForDelegate(
198:    function retrieveProxyContractAddress(
```


---

### State variable and function names [3]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol

```solidity
// private and internal `functions` should preppend with `underline`
155:    function createProxyDelegatorAndTransfer(
163:    function transferBetweenDelegators(
173:    function deployProxyDelegatorIfNeeded(
192:    function getBalanceForDelegate(
198:    function retrieveProxyContractAddress(
```
