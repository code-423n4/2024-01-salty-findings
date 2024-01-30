- In _depositLiquidityAndIncreaseShare(), Liquidity should reset approval after pools' addLiquidity().
```
tokenA.approve( address(pools), 0 );
tokenB.approve( address(pools), 0 );
```

- Suggest add return value check in _whitelist.add(poolID). If one pool has already added in whitelist, we can return back directly and save some gas.

```solidity
	function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
		{
		require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
		require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");

		bytes32 poolID = PoolUtils._poolID(tokenA, tokenB);

		// Add to the whitelist and remember the underlying tokens for the pool
		_whitelist.add(poolID);
		underlyingPoolTokens[poolID] = TokenPair(tokenA, tokenB);

		// Make sure that the cached arbitrage indicies in PoolStats are updated
		pools.updateArbitrageIndicies();

 		emit PoolWhitelisted(address(tokenA), address(tokenB));
		}
```
