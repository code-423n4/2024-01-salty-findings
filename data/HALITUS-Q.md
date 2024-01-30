# Reserve1 is unchecked in Pools.removeLiquidity()

Severity: LOW

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L187

## Vulnerability
As shown below, `reserves.reserve0 >= PoolUtils.DUST` is checked twice. One of the conditions should be `reserves.reserve1 >= PoolUtils.DUST`.

```solidity
function removeLiquidity() {
.
.
    // Make sure that removing liquidity doesn't drive either of the reserves below DUST.
    // This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
    require(
      (reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST),
      'Insufficient reserves after liquidity removal'
    );
.
.
}
```