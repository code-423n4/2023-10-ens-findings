## FOR LOOP OPTIMIZATION

### Description

Looking at the for loop [here](https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L85) it's possible to see that the value of 
`Math.max(sourcesLength, targetsLength)` is being calculated at every loop iteration.

Additionally we don't need to initialise the variable transferIndex to zero since this it's already the default value.

### Recommendation
We can optimize this as such:

~~~
uint maxLength = Math.max(sourcesLength, targetsLength);
for (uint transferIndex; transferIndex < maxLength ;transferIndex++) {
}
~~~