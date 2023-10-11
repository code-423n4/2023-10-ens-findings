**Issue #1**
**ERC20MultiDelegate._delegateMulti()**
Switch the for loop to a while loop, like in the following example:
```
uint transferIndex = 0;
uint maxItterations = Math.max(sourcesLength, targetsLength);
while (transferIndex < maxItterations) {
   // do stuff
   unchecked {
      ++i;
   }
}
```
This function will run out of gas before ```i``` can overflow.

**Issue #2**
**ERC20MultiDelegate.transferBetweenDelegators()**
Store the storage variable ```token``` in a stack variable. Reading a a storage variable that has been accessed costs 100 gas, while reading a stack variable costs under 10 gas.

**Issue #3**
**ERC20MultiDelegate._reimburse()**
Store the storage variable ```token``` in a stack variable. Reading a a storage variable that has been accessed costs 100 gas, while reading a stack variable costs under 10 gas.

**Issue #4**
**ERC20MultiDelegate.deployProxyDelegatorIfNeeded()**
Store the storage variable ```token``` in a stack variable. Reading a a storage variable that has been accessed costs 100 gas, while reading a stack variable costs under 10 gas.
