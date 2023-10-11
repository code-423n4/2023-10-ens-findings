# [G-1] Use custom errors instead of long string errors

In the codebase, the require statement is used with long error strings which consume a larger amount of gas. To avoid this, use custom errors. 

## Affected SLOC : 
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L74-L82

## Recommended Mitigation 

Use custom errors. For example, instead of : 

``` solidity 
        require(
            sourcesLength > 0 || targetsLength > 0,
            "Delegate: You should provide at least one source or one target delegate"
        );
```

Use: 

error provideAtLeastOneSourceOrTarget();
if(!(sourcesLength > 0 || targetsLength > 0)){
 revert provideAtLeastOneSourceOrTarget();
}

# [G-2] Set variables which won't be changed after contract initialization as immutable

The variable 'token' of type ERC20Votes can be changed to 'public immutable' type which will reduce gas consumption. 

## Affected SLOC : 
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L28

## Recommended Mitigation

Change the affected line of code with this : 

```solidity
    ERC20Votes public immutable token;
```

# [G-3] Use !=0 instead of >0 while comparing

The !=0 comparison operator uses less gas than >0 in our case since we use unsigned integers. We can replace them. 

## Affected SLOC : 

1. https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L74-L77
2. https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L110-L115

## Recommended Mitigation 

Change the operator as mentioned above.

# [G-4] Keep iteration of transferIndex in an unchecked block

The variable transferIndex is iterated in a for loop in the function _delegateMulti(). This can be kept inside an unchecked block since transferIndex won't go out of bounds and the value of Math.max(sourcesLength, targetsLength) won't change. 

## Affected SLOC : 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85-L89

## Recommended Mitigation : 

Add an unchecked block like this at the end of the for loop: 

```solidity
unchecked{
 ++transferIndex;
}
```

# [G-5] Redundant check in if - else - if construct

The check `transferIndex < targetsLength` is redundant and can be removed. 

## Affected SLOC : 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L104

## Recommended Mitigation Steps : 

Change the else if to else. 

# [G-6] Value computed needlessly inside comparison for for loop inside _delegateMulti() 

In the for loop inside the function _delegateMulti(), the value of `Math.max(sourcesLength, targetsLength)` is computed for each iteration even though it does not change. 

## Affected SLOC : 

https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L87

## Recommended Mitigation : 

Calculate the value earlier as per this example : 

```solidity
        uint256 maxValue = Math.max(sourcesLength, targetsLength);
```

and perform the check later on like this: 

```solidity
            transferIndex < maxValue;
```

