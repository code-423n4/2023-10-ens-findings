G[0-1]:Use proxyAddress.code.length==0 instead of assembly ERC20MultiDelegate.sol#L179-L185 
       besuse there is not any gas seving benfit useing assembly.

proof of concept:


contract GasTest is StdCheats, Test {
    Contract0 c0;
    Contract1 c1;
    address proxyAddress;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
        proxyAddress = makeAddr("alice");
    }

    function testGas() public view {
        c0.SolodityCode(proxyAddress);
        c1.assemblyCode(proxyAddress);
    }
}

contract Contract0 {
    function SolodityCode(address _proxyAddress) public view {
        if (_proxyAddress.code.length == 0) {}
    }
}

contract Contract1 {
    function assemblyCode(address _proxyAddress) public view {
        uint256 bytecodeSize;
        assembly {
            bytecodeSize := extcodesize(_proxyAddress)
        }

        if (bytecodeSize == 0) {}
    }
}

| test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|---------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                       | Deployment Size |     |        |     |         |
| 32287                                 | 191             |     |        |     |         |
| Function Name                         | min             | avg | median | max | # calls |
| SolodityCode                          | 235             | 235 | 235    | 235 | 1       |


| test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|---------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                       | Deployment Size |     |        |     |         |
| 32287                                 | 191             |     |        |     |         |
| Function Name                         | min             | avg | median | max | # calls |
| assemblyCode                          | 235             | 235 | 235    | 235 | 1       |

G[0-2]:Use cache instead calling max function in every loop.


     Cache” the array’s length for the loop condition or if condetion, 
     for loop and if condetion inside the _delegateMulti function uses Math.max(sourcesLength, targetsLength) in 
     its looping condition and Math.min(sourcesLength, targetsLength)in if condetion. 
     Consider declaring two memory object for Math.max(sourcesLength, targetsLength) and Math.min(sourcesLength, 
     targetsLength) , 
     and using that value in the looping condition instead of Math.max(sourcesLength, targetsLength) also for 
     Math.min(sourcesLength, targetsLength).
     this can save almost 100 gas for every loop.ERC20MultiDelegate.sol#L80,#L87,#L98  
     

     Total Instances: 3


     proof of concept:

    contract GasTest is Test {
       Contract0 c0;
       Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public view {
        c0.WithCache(1, 2);
        c1.WithoutCache(1, 2);
    }
}

contract Contract0 {
    function WithCache(uint256 a, uint256 b) public pure {
        uint256 max1 = Math.max(a, b);
        uint256 min1 = Math.min(a, b);
        for (uint256 transferIndex; transferIndex < max1; transferIndex++) {
            if (transferIndex < min1) {}
        }
    }
}

contract Contract1 {
    function WithoutCache(uint256 a, uint256 b) public pure {
        for (
            uint256 transferIndex;
            transferIndex < Math.max(a, b);
            transferIndex++
        ) {
            if (transferIndex < Math.min(a, b)) {}
        }
    }
}



| test/GasTest1.t.sol:Contract0 contract |                 |     |        |     |         |
|----------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                        | Deployment Size |     |        |     |         |
| 55705                                  | 310             |     |        |     |         |
| Function Name                          | min             | avg | median | max | # calls |
| WithCache                              | 654             | 654 | 654    | 654 | 1       |


| test/GasTest1.t.sol:Contract1 contract |                 |     |        |     |         |
|----------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                        | Deployment Size |     |        |     |         |
| 53705                                  | 300             |     |        |     |         |
| Function Name                          | min             | avg | median | max | # calls |
| WithoutCache                           | 865             | 865 | 865    | 865 | 1       |


G[0-3]:  Use do while loop instead  for loop 
         In function _delegateMulti because a do while loop will cost less gas since the condition is not being 
         checked for the first iteration.ERC20MultiDelegate.sol#L85-L116