### Report 1:
#### Incomplete State Boundary Implementation
maximumWhitelistedPools can be set below the current length of whitelisted pools in the PoolsConfig.sol contract this should be corrected as provided below.
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L87
```solidity
function changeMaximumWhitelistedPools(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maximumWhitelistedPools < 100)
                maximumWhitelistedPools += 10;
            }
        else
            {
            if (maximumWhitelistedPools > 20 )
                maximumWhitelistedPools -= 10;
+++            if( maximumWhitelistedPools <  _whitelist.length() ){
+++              revert("Error Message");
+++             }
            }

		emit MaximumWhitelistedPoolsChanged(maximumWhitelistedPools);
        }
```
###  Report 2:
#### Missing Validation check for PoolIds in PoolStats contract.
When a pool Id is invalid it returns INVALID_POOL_ID which represents type(uint64).max, however this return value was not checked at L88-L91 in the updateArbitrageIndicies() function. This would allow invalid pool during Arbitrage and could escalate to serious problems, The return value should be checked as adjusted below.
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L89-L91
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L71
```solidity
function _poolIndex( IERC20 tokenA, IERC20 tokenB, bytes32[] memory poolIDs ) internal pure returns (uint64 index)
	{
	bytes32 poolID = PoolUtils._poolID( tokenA, tokenB );

	for( uint256 i = 0; i < poolIDs.length; i++ )
		{
		if (poolID == poolIDs[i])
			return uint64(i);
		}

>>>	return INVALID_POOL_ID;
	}
```
```solidity
function updateArbitrageIndicies() public
	{
	bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

	for( uint256 i = 0; i < poolIDs.length; i++ )
	{
	bytes32 poolID = poolIDs[i];
	(IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);

	// The middle two tokens can never be WETH in a valid arbitrage path as the path is WETH->arbToken2->arbToken3->WETH.
	if ( (arbToken2 != _weth) && (arbToken3 != _weth) )
				{
>>>	uint64 poolIndex1 = _poolIndex( _weth, arbToken2, poolIDs );
>>>	uint64 poolIndex2 = _poolIndex( arbToken2, arbToken3, poolIDs );
>>>	uint64 poolIndex3 = _poolIndex( arbToken3, _weth, poolIDs );
+++     require ( poolIndex1 != INVALID_POOL_ID && poolIndex2 != INVALID_POOL_ID && poolIndex3 != INVALID_POOL_ID , "INVALID_POOL_ID" );
				...
		}
```
### Report 3:
#### Unused Variable
Possible Incomplete implementation in the performUpkeep(...) function of the RewardsEmitter.sol contracr. As noted in the code provided below, uint256 sum was declared and was continuously assigned a value of amountToAddForPool at every loop cycle, but `sum` was only assigned it was not used at all through the function implementation
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L115-L128
```solidity
function performUpkeep( uint256 timeSinceLastUpkeep ) external
   ...
>>> uint256 sum = 0;
for( uint256 i = 0; i < poolIDs.length; i++ )
	{
	bytes32 poolID = poolIDs[i];
		// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
	uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;

	// Reduce the pending rewards so they are not sent again
	if ( amountToAddForPool != 0 )
		{
		pendingRewards[poolID] -= amountToAddForPool;

>>>		sum += amountToAddForPool;
		}

	// Specify the rewards that will be added for the specific pool
	addedRewards[i] = AddedReward( poolID, amountToAddForPool );
	}

// Add the rewards so that they can later be claimed by the users proportional to their share of the StakingRewards derived contract.
stakingRewards.addSALTRewards( addedRewards );
}
```
### Report 4:
#### Comment Misinterpretation
The comment in the code provided below in the Pools.sol contract is to determine the value WETH would worth after swap with SwapAmountIn and not just proportionate value, therefore should be adjusted as provided below.
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L301-L313
```solidity
>>> // Determine the WETH equivalent of swapAmountIn of swapTokenIn
	function _determineSwapAmountInValueInETH(IERC20 swapTokenIn, uint256 swapAmountIn) internal view returns (uint256 swapAmountInValueInETH)
{
if ( address(swapTokenIn) == address(weth) )
	return swapAmountIn;

// All tokens within the DEX are paired with WETH (and WBTC) - so this pool will exist.
	(uint256 reservesWETH, uint256 reservesTokenIn) = getPoolReserves(weth, swapTokenIn);

if ( (reservesWETH < PoolUtils.DUST) || (reservesTokenIn < PoolUtils.DUST) )
	return 0; // can't arbitrage as there are not enough reserves to determine swapAmountInValueInETH

---    swapAmountInValueInETH = ( swapAmountIn * reservesWETH ) / reservesTokenIn;
+++    swapAmountInValueInETH = ( swapAmountIn * reservesWETH ) / ( reservesTokenIn + swapAmountIn );

if ( swapAmountInValueInETH <= PoolUtils.DUST )
	return 0;
}
```