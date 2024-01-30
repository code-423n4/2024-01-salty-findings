#LOW ISSUES REPORT 

## [L-01] Require xsalt for proposal can be so low

To submit a proposal in the Salty protocol, users must have staked a specific quantity of salt. The exact amount is determined by a percentage set by the DAO, multiplied by the total number of salt tokens staked in the protocol.

The issue arises when the total staked amount is very low, as it results in a low requirement of XSALT needed to make a proposal. 

```
function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
		{
		require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

		// The DAO can create confirmation proposals which won't have the below requirements
		if ( msg.sender != address(exchangeConfig.dao() ) )
			{
			// Make sure that the sender has the minimum amount of xSALT required to make the proposal
			uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

			require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

			uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
			require( userXSalt >= requiredXSalt, "Sender does not  
 have enough xSALT to make the proposal" );  <--------

			// Make sure that the user doesn't already have an active proposal
			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
			}
...
}
```
[[Link]](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81C2-L99C5)

## Impact
the dao can be spamed  making so many proposals for part of the users.

## Recomendation
Consider implementing a minimum threshold that users must stake to submit a proposal. 

## [L-02] missing <= in token.totalSupply() restriction 
One of the restrictions to whitelist a token is that the token supply cannot exceed uint112.max. As you can see in the code and comments:

```
function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID)
		{
		require( address(token) != address(0), "token cannot be address(0)" );
		require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision  <--------

		require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );
		require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );
		require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );

		string memory ballotName = string.concat("whitelist:", Strings.toHexString(address(token)) );

		uint256 ballotID = _possiblyCreateProposal( ballotName, BallotType.WHITELIST_TOKEN, address(token), 0, tokenIconURL, description );
		_openBallotsForTokenWhitelisting.add( ballotID );

		return ballotID;
		}
```
[[Link]](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L162C2-L177C4)

The problem is that the contract is not accepting tokens with type(uint112).max either.

##  Impact
Tokens with type(uint112).max totalSupply are incompatible with the project.

## Recomendation
Consider add the equal in the totalSuplly require `require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" );`

# [L-03] Unwhitelisted pools get loss the arbitrage profit.
Pools can be whitelisted or unwhitelisted via DAO proposals. When a pool is whitelisted, the DAO starts accruing arbitrage profits. These profits are reclaimed when the main performUpkeep function is called.

The problem is that when a pool is unwhitelisted, the profits accrued between the time of the last performUpkeep and the unwhitelisting transaction get lost in the protocol. 

```
function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID)
		{
		require( address(token) != address(0), "token cannot be address(0)" );
		require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision

		require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );
		require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );
		require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );

		string memory ballotName = string.concat("whitelist:", Strings.toHexString(address(token)) );

		uint256 ballotID = _possiblyCreateProposal( ballotName, BallotType.WHITELIST_TOKEN, address(token), 0, tokenIconURL, description );
		_openBallotsForTokenWhitelisting.add( ballotID );

		return ballotID;
		}
```

## Impact
The protocol loses arbitrage profits each time that a pool is unwhitelisted.

## Recomendation
Consider call `saltRewards.performUpkeep(poolIDs, profitsForPools);` each time that a pool is unwhitelisted.

# [L-04] User with large liqudity can be reward hunters.
The protocol is implementing a cooldown to protect against reward hunters:
```
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
		// Cooldown is specified to prevent reward hunting (ie - quickly depositing and withdrawing large amounts of liquidity to snipe rewards)  <-----
		// This call will send any pending SALT rewards to msg.sender as well.
		_decreaseUserShare( msg.sender, poolID, liquidityToWithdraw, true );

		emit LiquidityWithdrawn(msg.sender, address(tokenA), address(tokenB), reclaimedA, reclaimedB, liquidityToWithdraw);
		}
```
[[Link]](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L121C3-L140C4)


The cooldown, initially set to 1 hour and adjustable by the DAO, is primarily effective against flash loans but lacks effectiveness against users with significant liquidity.

## Impact
Users with significant liquidity have the ability to increase their share (via the liquidity contract) one minute before the rewards are distributed and subsequently decrease their shares one hour later, successfully claiming the rewards.

## Recomendation
Consider increasing the default modificationCooldown to restrict users with significant liquidity from exploiting the reward distribution.

