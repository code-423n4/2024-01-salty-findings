## [GAS-1] The Ownable library is unnecessary on the `USDS.sol` contract.
There is 1 instance of this
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L14

The `onlyOwner` modifier of the `ownable` contract was used only once in the  function that is to be executed only once at deployment, then ownership is renounced in the same function. 

Since the below function is the only function where `onlyOwner` modifier is used and it is executed only once at deployment according to the comment above it, the code in the `setCollateralAndLiquidity(...)` can be implemented in the constructor since the constructor is run only once and can only be by the deployer.

```
File: src/stable/USDS.sol
28: // This will be called only once - at deployment time
29:	function setCollateralAndLiquidity( ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
30:		{
31:		collateralAndLiquidity = _collateralAndLiquidity; //@audit set this in the constructor since it will be called at deployment.

32:		// setCollateralAndLiquidity can only be called once
33:		renounceOwnership();//@audit ownable is unnecessary.
34:		}
```

Impact:
Reduces deployment gas cost because the `Ownable` contract code is removed.

Recommendation:
Implement the code in the `setCollateralAndLiquidity(...)` in the constructor of the contract this way:

```
constructor(ICollateralAndLiquidity _collateralAndLiquidity) ERC20( "USDS", "USDS" ) {
   collateralAndLiquidity = _collateralAndLiquidity;// also emit event.
}
```

The above will remove the need for the `setCollateralAndLiquidity(...)` and the `Ownable` contract codes.