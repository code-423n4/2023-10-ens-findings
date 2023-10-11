**Low Risk Issues**
----------------------------------

**[LOW-1] ```_processDelegation()``` can be temporarily DoS if a user sets a target > ((2^160) - 1) :**
Users need to check their parameters while using ```delegateMulti()``` otherwise they could face a temporary DoS.

The [L-10] issue in the automated report expresses a similar cause but does not clearly specify the impact of this bug.

**Proof of Concept:**
The multi-delegate function accepts uint256[] arrays instead of address[] arrays:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L57
```
function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external {
        _delegateMulti(sources, targets, amounts);
    }
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L65
```
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal {
        uint256 sourcesLength = sources.length;
        uint256 targetsLength = targets.length;
        uint256 amountsLength = amounts.length;

        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );

        require(
            Math.max(sourcesLength, targetsLength) == amountsLength,
            "Delegate: The number of amounts must be equal to the greater of the number of sources or targets"
        );

        // Iterate until all source and target delegates have been processed.
        for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
            address source = transferIndex < sourcesLength
                ? address(uint160(sources[transferIndex]))
                : address(0);
            address target = transferIndex < targetsLength
                ? address(uint160(targets[transferIndex]))
                : address(0);
            uint256 amount = amounts[transferIndex];

            if (transferIndex < Math.min(sourcesLength, targetsLength)) {
                // Process the delegation transfer between the current source and target delegate pair.
                _processDelegation(source, target, amount);
            } else if (transferIndex < sourcesLength) {
                // Handle any remaining source amounts after the transfer process.
                _reimburse(source, amount);
            } else if (transferIndex < targetsLength) {
                // Handle any remaining target amounts after the transfer process.
                createProxyDelegatorAndTransfer(target, amount);
            }
        }

        if (sourcesLength > 0) {
            _burnBatch(msg.sender, sources, amounts[:sourcesLength]);
        }
        if (targetsLength > 0) {
            _mintBatch(msg.sender, targets, amounts[:targetsLength], "");
        }
    }
```
The ```_delegateMulti()``` function has no check if each ```source``` or ```target``` are always <= ((2^160)-1). This would create a temporary problem for the users where they would face a temporary DoS when using ```_processDelegation()``` function. Let us break it down step by step:

Step-1: A user (with address X) calls the ```delegateMulti()``` function with the following parameters:
```sources``` = [] (empty array as the user delegates 1st time)
```targets``` = [7237005577332262213973186563042994240829374041602535252466099000494570602501] (= 0x1000000000000000000000000000000000000000000000000000000000000005)
```amounts``` = [2e18]

Step-2: A proxy contract is created and it points (delegates) to the delegatee address = address(uint160(7237005577332262213973186563042994240829374041602535252466099000494570602501)) = 0x0000000000000000000000000000000000000005

Step-3: The ERC20Votes tokens are transferred from the user to the proxy. At last the mapping in the ERC20MultiDelegate.sol is updated as:
7237005577332262213973186563042994240829374041602535252466099000494570602501 --> X --> 2e18

Now, suppose X wants to transfer delegation to some other address. X would call ```delegateMulti()``` with their chosen ```targets``` (of length = 1), ```sources``` as [7237005577332262213973186563042994240829374041602535252466099000494570602501] and ```amounts``` as [2e18]. Let us see what happens step by step:

Step-1: The delegation is first transferred to ```targets[0]``` from ```sources[0]```. This happens in:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L100

The parameters for ```_processDelegation()``` function are now:
```source``` = address(uint160(7237005577332262213973186563042994240829374041602535252466099000494570602501)) = 0x0000000000000000000000000000000000000005
```target``` is the address in the ```targets[0]```
```amount``` = 2e18

Step-2: The ```_processDelegation()``` function is called:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L124
```
function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
 At LOC-129, ```getBalanceForDelegate(source)``` is called:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L192
```
function getBalanceForDelegate(
        address delegate
    ) internal view returns (uint256) {
        return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
    }
```
We should focus on LOC-195:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L195
```
return ERC1155(this).balanceOf(msg.sender, uint256(uint160(delegate)));
```
Here, ```uint256(uint160(delegate))``` is used. Applying this in our case, we would have:
uint256(uint160(0x0000000000000000000000000000000000000005)) = 5
However, the mapping in ERC20MultiDelegate.sol has 0 tokens corresponding to id = 5 for X:
5 --> X --> 0
Hence, 0 is returned. Now, at LOC-131:
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131
```
assert(amount <= balance);
```
the transaction would revert as ```amount``` = 2e18 and ```balance``` would be 0.

**Note:**

One way for the user to mitigate this (after the user has mistakenly used a uint256 target > (2^160) - 1) is that they would reimburse their tokens and re-delegate again with the correct target. However, this would hamper the user experience.

The likelihood is LOW (assuming the users would be well aware of how to use the ```_delegateMulti()``` function).
The severity seems MEDIUM (as DoS for a specific function).

***Recommended Mitigation Steps:***
In the function
```
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal
```
check if targets and sources are <= (2^160) - 1.

**[LOW-2]: Ambiguity for users who have already delegated tokens in the ERC20Votes.sol**

There would be a problem for users who try to delegate through the ERC20MultiDelegate.sol as well as its ERC20Votes.sol.

**Proof of Concept:**
Suppose a user X has already delegated to another user Y all of their balance through the ```delegate(address delegatee)``` function of the ERC20Votes.sol.
Now, suppose they want to transfer their voting power to Z using ERC20MultiDelegate.sol.  
If they tried to use ERC20MultiDelegate.sol (integrated with the ERC20Votes.sol) for the first time and called
```
function delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) external
```
 with ```sources``` as [uint256(uint160(Y))], they would face a DoS as the ERC20MultiDelegate.sol would have the mapping:
uint256(uint160(Y)) --> X --> 0 (that is, balance is 0 and thus no positive amount can be transferred to another delegatee).

This would happen when the ```assert(amount <= balance)``` executes in the function
```
function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal
```
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131
```
function _processDelegation(
        address source,
        address target,
        uint256 amount
    ) internal {
        uint256 balance = getBalanceForDelegate(source);

        assert(amount <= balance);

        deployProxyDelegatorIfNeeded(target);
        transferBetweenDelegators(source, target, amount);

        emit DelegationProcessed(source, target, amount);
    }
```
The main assumption behind this is that the user X interacts with the ERC20MultiDelegate.sol as well as its ERC20Votes.sol (as this is what we have in-scope) and doesn't know that mapping in ERC20MultiDelegate.sol and its ERC20Votes.sol are maintained separately. It is required that the user must know in prior that even if they have delegated through ERC20Votes.sol, they would still use an empty ```sources``` array in the ERC20MultiDelegate.sol, which would seem ambiguous.

The likelihood is LOW (assuming the users are aware of the above mentioned assumption).
The severity is LOW (as the user would still be able to execute operations using empty ```sources``` array).

***Recommended Mitigation Steps:***
The mitigation step would require to implement a facility in the function
```
function _delegateMulti(
        uint256[] calldata sources,
        uint256[] calldata targets,
        uint256[] calldata amounts
    ) internal
```
to check if the msg.sender has already delegated to a address in the ERC20Votes.sol (given the msg.sender calls _delegateMulti for the first time), OR, disable the facility to delegate in the ERC20Votes.sol.