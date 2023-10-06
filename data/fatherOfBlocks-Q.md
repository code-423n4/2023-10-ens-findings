**contracts/ERC20MultiDelegate.sol**
- L206 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L151 - According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern

- L48 - A variable is set in storage that cannot be modified later, therefore it would be beneficial to add a validation of _token != 0x.

- L131 - Input validation is performed with an asserts, this breaks the standard of using errors handling, require should be used to validate inputs.
https://www.linkedin.com/pulse/require-vs-revert-assert-solidity-arslan-maqbool/
