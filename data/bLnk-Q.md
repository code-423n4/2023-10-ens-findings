Due to downcasting in _delegateMulti function the same address can be generated from 2 different inputs.

Code:
91:                 ? address(uint160(sources[transferIndex]))

94:                 ? address(uint160(targets[transferIndex]))

Here if on transferIndex position the number is 1 or 2 ** 160 + 1 then the generated address will be the same. 

