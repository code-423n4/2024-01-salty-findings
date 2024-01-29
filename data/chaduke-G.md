G1.  ``findLiquidatableUsers()`` resizes array ``liquidatableUsers`` into array ``resizedLiquidatableUsers`` by copying element-wise. This is a waste of gas. The resizing can be done by assembly in the original array ``liquidatableUsers`` without copying.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L311-L342](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L311-L342)

Mitigation: Simply use assembly to resize array ``liquidatableUsers``

```javascript
 assembly { mstore(liquidatableUsers, count) }
 return liquidatableUsers;
```