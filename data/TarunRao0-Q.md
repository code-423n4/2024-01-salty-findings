## [L-6] Additional Instance of Division before Multiplication 

**Description** :

Found another instance where multiplication is performed on the result of division in `PoolMath::_zapSwapAmount` 

**Impact** : 
Precision loss from rounding errors 

**Proof of Code** :

```javascript
uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
```

*github permalink*:

https://github.com/code-423n4/2024-01-salty/blob/6a9942091392866c674104dd4b7a12a2d14fcbf5/src/pools/PoolMath.sol#L192

