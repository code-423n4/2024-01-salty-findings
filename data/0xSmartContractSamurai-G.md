CHECKS TO HELP ME:

- [ ] Gas Optimization checks

    - [ ] Cache read variables in memory
    - [ ] Use `calldata` instead of `memory` if not modifying the function parameter passed
    - [ ] Don't call `view` function inside of another function
    - [ ] Use `++i` instead of `i++` (gas consumption order `i+=1` > `i=i+1` > `i++` > `++i`). Same for `--i`.
    - [ ] Use `unchecked` to not check for integer overflow and underflow if not required
    - [ ] Use `constant` and `immutable` for variables that don't change
    - [ ] Use `revert` instead of `require`
    - [ ] Create custom errors rather than `revert()`/`require()` strings to save gas
    - [ ] Put `require` statements on top in a function
    - [ ] Use `indexed` if less than three arguments are there in events for faster access
    - [ ] Use `!= 0` instead of `> 0` for unsigned integer comparison
    - [ ] `internal` functions not called by the contract should be removed
    - [ ] Cache array length outside of loop
    - [ ] Use `bytes` instead of `string`. Bytes constants are more efficient than string constants
    - [ ] Use external function modifier.
    - [ ] Use full 256 bit types unless packing with other variables.
    - [ ] Splitting `require` statements that use && saves gas.
    - [ ] State variables can be packed into fewer storage slots.
    - [ ] Use scratch space for keccak.
    - [ ] No Need to Allocate Unused Variable.
    - [ ] Use basis points for ratios.
    - [ ] Skip initializing default values.
    - [ ] Non-strict inequalities are cheaper than strict ones
    - [ ] Usage of `uint8` may increase gas cost
    - [ ] Use bytesN instead of bytes[]
    - [ ] Inline a modifier that’s only used once.
    - [ ] Inverting the condition of an [if-else-statement](https://gist.github.com/IllIllI000/44da6fbe9d12b9ab21af82f14add56b9) wastes gas.
    - [ ] [Consider having short revert strings.](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings)
    - [ ] Remove public visibility from constant variables
    - [ ] Emitting storage values instead of memory calldata ones does cost more gas
    - [ ] `MULMOD` opcode is cheaper than `MUL` and `MOD` opcodes when used together
    - [ ] Use double if statements instead of &&
    - [ ] Don’t call a function when initializing an immutable variable
    - [ ] Use assembly to write address storage values
    - [ ] Sort Solidity operations using short-circuit mode

=======
=======

0. `for( uint256 i = 0; i < 8; i++ )` can be gas optimized:

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/arbitrage/ArbitrageSearch.sol#L115

```diff
- for( uint256 i = 0; i < 8; i++ )
+ for( uint256 i; i < 8; ++i ) /// and could even do `uncheck {++i;}` instead below this line at end of for loop logic.
```

1. should cache the length of the state variable struct, as well as some other gas optimizations.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/dao/Proposals.sol#L423

```diff
+ uint256 openBallotsTokenWhitelistLength = _openBallotsForTokenWhitelisting.length();;
- for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
+ for( uint256 i; i < openBallotsTokenWhitelistLength; ++i )
```

2. can do `msb = msb + 128` instead as it uses less gas...same for all below...

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/pools/PoolMath.sol#L117-L127

```solidity
    function _mostSignificantBit(uint256 x) internal pure returns (uint256 msb)
    	{
        if (x >= 2**128) { x >>= 128; msb += 128; } 
        if (x >= 2**64) { x >>= 64; msb += 64; }
        if (x >= 2**32) { x >>= 32; msb += 32; }
        if (x >= 2**16) { x >>= 16; msb += 16; }
        if (x >= 2**8) { x >>= 8; msb += 8; }
        if (x >= 2**4) { x >>= 4; msb += 4; }
        if (x >= 2**2) { x >>= 2; msb += 2; }
        if (x >= 2**1) { x >>= 1; msb += 1; }
	    }
```


