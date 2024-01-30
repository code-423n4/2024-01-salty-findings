The function _withdrawLiquidityAndClaim not followed Checks effects interactions, may lead to a read-only reentrancy attacks

```
src/staking/Liquidity.sol

	// Withdraw specified liquidity, decrease the user's liquidity share and claim any pending rewards.
    function _withdrawLiquidityAndClaim( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToWithdraw, uint256 minReclaimedA, uint256 minReclaimedB ) internal returns (uint256 reclaimedA, uint256 reclaimedB)
		{
		bytes32 poolID = PoolUtils._poolID( tokenA, tokenB );
		require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

		// Remove the amount of liquidity specified by the user.
		// The liquidity in the pool is currently owned by this contract. (external call)
		(reclaimedA, reclaimedB) = pools.removeLiquidity( tokenA, tokenB, liquidityToWithdraw, minReclaimedA, minReclaimedB, totalShares[poolID] );

		// Transfer the reclaimed tokens to the user
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		// Reduce the user's liquidity share for the specified pool so that they receive less rewards.
		// Cooldown is specified to prevent reward hunting (ie - quickly depositing and withdrawing large amounts of liquidity to snipe rewards)
		// This call will send any pending SALT rewards to msg.sender as well.
		_decreaseUserShare( msg.sender, poolID, liquidityToWithdraw, true );

		emit LiquidityWithdrawn(msg.sender, address(tokenA), address(tokenB), reclaimedA, reclaimedB, liquidityToWithdraw);
		}
    
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L137

The code logic does not meet CEI, Since erc777 is fully compatible with erc20, if finalized an erc777 token to whitelist token, Malicious users can potentially profit from read-only reentrancy attacks