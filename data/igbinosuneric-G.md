## UNUSED NAMED RETURNS

Using both named returns and a return statement isn't necessary. Removing unused named return variables can reduce gas usage and improve code clarity.

https://github.com//code-423n4/2024-01-salty/blob/main/src/dev/Utils.sol#L255-L282
```
function quoteAmountOut( IPools pools, IERC20[] memory tokens, uint256 amountIn ) external view returns (uint256 amountOut)
		{
		require( tokens.length >= 2, "Must have at least two tokens swapped" );

		IERC20 tokenIn = tokens[0];
		IERC20 tokenOut;

		for( uint256 i = 1; i < tokens.length; i++ )
			{
			tokenOut = tokens[i];

			(uint256 reserve0, uint256 reserve1) = pools.getPoolReserves(tokenIn, tokenOut);

			if ( reserve0 <= PoolUtils.DUST || reserve1 <= PoolUtils.DUST || amountIn <= PoolUtils.DUST )
				return 0;

			uint256 k = reserve0 * reserve1;

			// Determine amountOut based on amountIn and the reserves
			reserve0 += amountIn;
			amountOut = reserve1 - k / reserve0;

			tokenIn = tokenOut;
			amountIn = amountOut;
			}

		return amountOut;
		}

```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dev/Utils.sol#L287-L314

```
function quoteAmountIn(  IPools pools, IERC20[] memory tokens, uint256 amountOut ) external view returns (uint256 amountIn)
		{
		require( tokens.length >= 2, "Must have at least two tokens swapped" );

		IERC20 tokenOut = tokens[ tokens.length - 1 ];
		IERC20 tokenIn;

		for( uint256 i = 2; i <= tokens.length; i++ )
			{
			tokenIn = tokens[ tokens.length - i];

			(uint256 reserve0, uint256 reserve1) = pools.getPoolReserves(tokenIn, tokenOut);

			if ( reserve0 <= PoolUtils.DUST || reserve1 <= PoolUtils.DUST || amountOut >= reserve1 || amountOut < PoolUtils.DUST)
				return 0;

			uint256 k = reserve0 * reserve1;

			// Determine amountIn based on amountOut and the reserves
			// Round up here to err on the side of caution
			amountIn = Math.ceilDiv( k, reserve1 - amountOut ) - reserve0;

			tokenOut = tokenIn;
			amountOut = amountIn;
			}

		return amountIn;
		}
```