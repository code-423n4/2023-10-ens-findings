1)Line 86 

for (
            uint transferIndex = 0;
            transferIndex < Math.max(sourcesLength, targetsLength);
            transferIndex++
        )

2)Line 179

uint bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(proxyAddress)
        }


The uint data type should have a suffix for example uint256 or else such as the code is more readable, but in solidity documentation the uint default definition is uint256, however is important to append a suffix, because if this "uints" can be lower than 256 you could save a little of gas as well.