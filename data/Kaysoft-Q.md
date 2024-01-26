## [L-1] Lack of `deadline` check on add and removing liquidity.
There are 2 instances of this

- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L140
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L170

The `ensureNotExpired` modifier is used on the `Pool.swap()` function but not used on the `addLiquidity(...)` and `removeLiquidity(...)` functions of Pool.sol

The `ensureNotExpired` modifier is used to protect transactions from being kept by the miners till it is profitable for them before execution especially in unstable market swings.

```
File: Pool.sol
modifier ensureNotExpired(uint deadline)
		{
		require(block.timestamp <= deadline, "TX EXPIRED");
		_;
		}
```

```
File: Pool.sol
function addLiquidity( ... ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity){
...
}
```
However, the `ensureNotExpired` modifier is not added to the `addLiquidity(...)` and `removeLiquidity(...)` functions.

Impact: `removeLiquidity` and `addLiquidity` transactions can be "saved for later" for as long as possible until it incurs enough loss based on slippage.

Recommendation: add a `deadline` parameter and the `ensureNotExpired` modifier to the `addLiquidity(...)` and `removeLiquidity(...)` functions as it is done on the `swap(...)` function. 
```
function addLiquidity( ..., deadline) external nonReentrant ensureNotExpired(deadline) returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity){
...
}
```


## [L-2] Use the available `private` `visibility specifier` instead of creating the `onlySameContract` modifier

There are 11 instances of this
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L95C2-L112C58
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L145C2-L231C43

According the to Solidity docs:
> private function: only visible in the current contract

And here is the `onlySameContract` modifier:
```
File:Upkeep.sol
modifier onlySameContract()
		{
    	require(msg.sender == address(this), "Only callable from within the same contract");
    	_;
		}
```
It is better to user the `private` keyword instead of creating and using a new modifier that achieves the same aim as the `private` visibility specifier.

Impact: More unnecessary code when `private` can be used.

Recommedation:
Consider using `private` visibility specifier on functions `step1(...)` to `step11(...)`.
```diff
--     function step1() public onlySameContract
++     function step1() private
		{
		collateralAndLiquidity.liquidizer().performUpkeep();
		}
```



## [L-3] The Ownable library is unnecessary on the `USDS.sol` contract.
There are 2 instance of this
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L14
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L21

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

The same applies to the `Liquizer.sol` contract and the `Liquidizer.sol#setContracts(...)` function.
- https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L21


Impact:
Adds more unnecessary codes that comes with the `Ownable` contract to the `USDC.sol` contract increasing complexity and deployment gas cost.

Recommendation:
Implement the code in the `setCollateralAndLiquidity(...)` in the constructor of the contract this way:

```
constructor(ICollateralAndLiquidity _collateralAndLiquidity) ERC20( "USDS", "USDS" ) {
   collateralAndLiquidity = _collateralAndLiquidity;// also emit event.
}
```

The above will remove the need for the `setCollateralAndLiquidity(...)` and the `Ownable` contract codes.



