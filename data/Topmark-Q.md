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