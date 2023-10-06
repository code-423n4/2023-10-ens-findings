### Description
While looping through arrays, it is preferred to ++i instead of i++.

i++ and ++i are designed to work the same way, but i++ is slightly more expensive due to the way it works.

When i++ is complied, it results as below

j = i;
i = i + 1;
return j
compared to that, ++i results as below

i = i + 1;
return i;
Since this is a loop, the number iterations has impact on the gas consumption. Optimization where possible is already recommended.

Reference:
https://ethereum.stackexchange.com/questions/133161/why-does-i-cost-less-gas-than-i

Issue code:
```
  for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        ) {
```
Fixed code:
```
 for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            ++transferIndex
        ) {
```