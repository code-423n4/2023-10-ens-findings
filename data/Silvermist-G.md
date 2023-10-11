| Number | Details | Instances |
|-----------|-----------|-----------|
| G-01 | [Can make the variable outside the loop to save gas](#gas-1) | 1|
| G-02 | [Can make the variable outside the if statment to save gas](#gas-2) | 1|


G-01 Can make the variable outside the loop to save gas
Consider checking the max value before the loop which gonna save gas
```        
for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L85C9-L89C10


G-02 Can make the variable outside the if statment to save gas
Consider checking the max value before the if which gonna save gas
```        
if (transferIndex < Math.min(sourcesLength, targetsLength)) 
```
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/contracts/ERC20MultiDelegate.sol#L98C1-L99C1
