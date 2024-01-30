# Quality Assurance Report


| QA Issues | Issues                                         | Instances |
| --------- | ---------------------------------------------- | --------- |
| [[L-01](#l-01-in-poolsremoveliquidity-function-reservesreserve0-used-twice-in-one-condition-but-reservesreserve1-not-checked)] | In `Pools::removeLiquidity` function `reserves.reserve0` used twice in one condition but `reserves.reserve1` not checked. | 1 |
| [[L-02](#l-02-check-value-before-dividing-by-zero)] | Check value before dividing by zero | 8 |
| [[L-03](#l-03-unsafe-downcasting)] | Unsafe downcasting | 2 |
| [[L-04](#l-04-use-owanble2step-and-override-renounce-ownership-so-owner-cant-renounce-it)] | Use owanble2step and override renounce ownership so owner can't renounce it. | 2 |
| [[L-05](#l-05-check-return-value-of-approve)] | check return value of approve | 2 |
| [[L-06](#l-06-after-minting-one-time-100-million-salt-tokens-there-is-no-way-to-mint-again-only-burn)] | After minting one time 100 Million Salt tokens there is no way to mint again only burn. | 1 |
| [[N-01](#n-01-refactor-requireif-statement-to-modifier-if-it-is-used-more-than-once)] | Refactor require()/if() statement to modifier if it is used more than once | 2 |
| [[N-02](#n-02-not-used-return-name-variable)] | Not used return name variable | 1 |
| [[N-03](#n-03-use-uint256-instead-of-uint)] | Use `uint256` instead of `uint` | 1 |
| [[N-04](#n-04-typo)] | Typo | 1 |
# Low-Severity

## [L-01] In `Pools::removeLiquidity` function `reserves.reserve0` used twice in one condition but `reserves.reserve1` not checked.
It can lead to continuing a transaction even though `reserves.reserve1` is less than DUST. It is not intended flow of the function. This is to ensure that ratios remain relatively constant even after a maximum withdrawal. But this will not be true. So better would be to `reserves.reserve1` is not less than DUST.

```diff
File : src/pools/Pool.sol

185: // Make sure that removing liquidity doesn't drive either of the reserves below DUST.
186:		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
- 187:        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
+ 187:        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L185C3-L187C146

## [L-02] Check value before dividing by zero

Checking for zero before performing a division operation is crucial to prevent division by zero errors. If you attempt to divide a number by zero in most programming languages, it will result in an error or an exception. In Solidity and smart contract development, encountering such errors can have severe consequences, potentially leading to a halt in the execution of the smart contract or unexpected behavior.

_8 instances_

### Check `totalLiquidity` 

```solidity
File : src/pools/Pools.sol

179:		reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
180:		reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L179C1-L180C75

### Check `reserve1` and `reserve0`

```solidity
File : src/pools/Pools.sol

257:    if (flipped)
258:        	{
259:			reserve1 += amountIn;
260:			amountOut = reserve0 * amountIn / reserve1;
261:			reserve0 -= amountOut;
262:        	}
263:     else
264:        	{
265:			reserve0 += amountIn;
266:			amountOut = reserve1 * amountIn / reserve0;
267:			reserve1 -= amountOut;
268:        	}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L257C9-L268C11

### Check `p`

```solidity
File : src/price_feed/CoreUniswapFeed.sol

72:      return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L72C3-L72C50

### Check `totalProfits`

```solidity
File : src/rewards/SaltRewards.sol

67:     uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L67C4-L67C92

### Check `totalCollateralShares`

```solidity
File : src/stable/CollateralAndLiquidity.sol

232:		uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares;
233:		uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232C1-L233C85

### Check `unstakeRange`

```solidity
File : src/staking/Staking.sol

211:    return numerator / ( 100 * unstakeRange );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L211C6-L211C48

### Check `totalShares[poolID]`

```solidity
File : src/staking/StakingRewards.sol

114:    uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L114C3-L114C99

## [L-03] Unsafe downcasting
It can result in wrong calculations and data loss.
**Note: Missing by bot-report**

```solidity
File : src/pools/Pools.sol

90:     function _addLiquidity( bytes32 poolID, uint256 maxAmount0, uint256 maxAmount1, uint256 totalLiquidity ) internal returns(uint256 addedAmount0, uint256 addedAmount1, uint256 addedLiquidity)
91: 		{
...    
100:			reserves.reserve0 += uint128(maxAmount0);
101:			reserves.reserve1 += uint128(maxAmount1);

125:  		    reserves.reserve0 += uint128(addedAmount0);
126: 		    reserves.reserve1 += uint128(addedAmount1);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L100C1-L101C45, https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L125C1-L126C46

## [L-04] Use Ownable2step and override renounceOwnership so owner can't renounce it.

```solidity
File : src/ExchangeConfig.sol

contract ExchangeConfig is IExchangeConfig, Ownable

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L13C1-L13C52

## [L-05] check return value of approve

**Note: Missing by bot-report**

```solidity
File : src/Upkeep.sol

67:     constructor( IPools _pools, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IDAOConfig _daoConfig, IStableConfig _stableConfig, IPriceAggregator _priceAggregator, ISaltRewards _saltRewards, ICollateralAndLiquidity _collateralAndLiquidity, IEmissions _emissions, IDAO _dao )
68: 		{
...

91:      weth.approve( address(pools), type(uint256).max );      

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L91C3-L91C53

## [L-06] After minting one time 100 Million Salt tokens there is no way to mint again only burn.
All Salt tokens will be burned result in disrupting the functionality of protocol.
```solidity
File : src/Salt.sol

constructor()
		ERC20( "Salt", "SALT" )
		{
		_mint( msg.sender, INITIAL_SUPPLY );
        }


	// SALT tokens will need to be sent here before they are burned.
	// There should otherwise be no SALT balance in this contract.
    function burnTokensInContract() external
    	{
    	uint256 balance = balanceOf( address(this) );
    	_burn( address(this), balance );

    	emit SALTBurned(balance);
    	}
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol

# Non-Critical

## [N-01] Refactor require()/if() statement to modifier if it is used more than once

**Note: Missing by bot-report**

_2 instances_

```solidity
File : src/dao/Proposals.sol

224:     require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

233:     require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L224C3-L224C85, https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L233C3-L233C85

## [N-02] Not used return name variable

**Note: Missing by bot-report**

```solidity
File : src/dao/Proposals.sol

162:	function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID)
...
176:    return ballotID;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L176C3-L176C19

## [N-03] Use `uint256` instead of `uint`

```solidity
File : src/staking/Liquidity.sol

40:      modifier ensureNotExpired(uint deadline)

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L40C2-L40C42

## [N-04] Typo

```solidity
File : src/Upkeep.sol

231:	function step11() public onlySameContract
232:		{
233:		uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
234:
235:		// teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
236:		VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
237:
238:		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
239:		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L231C1-L239C4

```diff
File : src/Upkeep.sol

	function step11() public onlySameContract
		{
-		uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
+		uint256 releasableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));

		// teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
		VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));

-		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
+		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releasableAmount );
		}

```