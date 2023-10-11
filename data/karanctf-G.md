## Use `amountsLength` instead of using max() on every itteration.

**summary:** As in above line the invariant is confirmed with require statement that 
https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80
`max(sourcesLength, targetsLength) == amountsLength`
So instead of using  `transferIndex < math.max` use amountsLength
```diff

require(

Math.max(sourcesLength, targetsLength) == amountsLength,

"Delegate: The number of amounts must be equal to the greater of the number of sources or targets"

);

  

// Iterate until all source and target delegates have been processed.

for (

uint transferIndex = 0;
++ transferIndex < amountsLength;
-- transferIndex < Math.max(sourcesLength, targetsLength); 

transferIndex++

) {
```