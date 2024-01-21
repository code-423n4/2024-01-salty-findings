## Summary

The function `burnTokensInContract()` in `SALT.sol` calls `_burn()` and emits an Event even if the balance is zero (there are no tokens to burn)

## Impact

Useless operations can be avoided by performing them only if the balance is > 0

## Proof of Concept

The function body shows how there are no controls before calling `_burn()`:

```
function burnTokensInContract() external
	{
	uint256 balance = balanceOf( address(this) );
	_burn( address(this), balance );

	emit SALTBurned(balance);
	}
```

## Tools Used

Manual review

## Recommended Mitigation Steps

Call `_burn()` and emit Event only when balance is > 0

```diff
diff --git a/src/Salt.sol b/src/Salt.sol
index da51dc1..1bd1510 100644
--- a/src/Salt.sol
+++ b/src/Salt.sol
@@ -25,10 +25,13 @@ contract Salt is ISalt, ERC20
     function burnTokensInContract() external
     	{
     	uint256 balance = balanceOf( address(this) );
+       
+       if (balance > 0) {
     	_burn( address(this), balance );
 
     	emit SALTBurned(balance);
     	}
+       }
```
