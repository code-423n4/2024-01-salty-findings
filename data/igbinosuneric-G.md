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

## REDUNDANT ARRAY INITIALIZATION

Declaring Arrays in Solidity by default assigns all the elements to 0 values therefore any attempt to overwrite them again with 0 is redundant.

The array secondsAgo was found to be affected on Line 52

https://github.com//code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54-L54

```
secondsAgo[1] = 0; // to (now)
```

## CONSTANT STATE VARIABLES
The contract has defined state variables whose values are never modified throughout the contract.
The variables whose values never change should be marked as constant to save gas.

https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L45-L45

```
EnumerableSet.UintSet private _allOpenBallots;
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L48-L48
```
EnumerableSet.UintSet private _openBallotsForTokenWhitelisting;
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L29-L29
```
EnumerableSet.Bytes32Set private _whitelist;
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L23-L23
```
EnumerableSet.AddressSet private _authorizedUsers;
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L46-L46
```
EnumerableSet.AddressSet private _walletsWithBorrowedUSDS;
```

## LONG REQUIRE/REVERT STRINGS
The require() and revert() functions take an input string to show errors if the validation fails.
This strings inside these functions that are longer than 32 bytes require at least one additional MSTORE, along with additional overhead for computing memory offset, and other parameters.

https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L83-L83

```
require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L102-L102
```
require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
```
## ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)

https://github.com//code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L87-L87
```
for( uint256 i = 0; i < addedRewards.length; i++ )
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L129-L129
```
for( uint256 i = 0; i < poolIDs.length; i++ )
```
## VARIABLES DECLARED BUT NEVER USED
The contract PoolUtils has declared a variable STAKED_SALT but it is not used anywhere in the code. This represents dead code or missing logic.

Unused variables increase the contract's size and complexity, potentially leading to higher gas costs and a larger attack surface.
https://github.com//code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L16-L16
```
bytes32 constant public STAKED_SALT = bytes32(0);
```
## UNUSED IMPORTS
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L14-L14
```
import "./interfaces/IDAO.sol";
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L11-L11
```
import "../staking/interfaces/IStaking.sol";
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L18-L18
```
import "../Upkeep.sol";
```
https://github.com//code-423n4/2024-01-salty/blob/main/src/dao/interfaces/IDAO.sol#L4-L4
```
import "../../rewards/interfaces/ISaltRewards.sol";
```
