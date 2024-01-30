- Inconsistency of Code Implementation and the Code Comments

There is a code comment in the PoolUtils.sol that explains how token reserves less than DUST (100) would be disregarded.

```
// Token reserves less than dust are treated as if they don't exist at all.
// With the 18 decimals that are used for most tokens, DUST has a value of 0.0000000000000001

uint256 constant public DUST = 100;
```

However, in Abritrage.sol, the _bisectionSearch function implements a check that requires that token reserves is less than or equal to DUST (100).

```
if ( reservesA0 <= PoolUtils.DUST || reservesA1 <= PoolUtils.DUST || reservesB0 <= PoolUtils.DUST || reservesB1 <= PoolUtils.DUST || reservesC0 <= PoolUtils.DUST || reservesC1 <= PoolUtils.DUST )
return 0;
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L107

Remediation:
Correct checks of token reserves to be less than DUST only.