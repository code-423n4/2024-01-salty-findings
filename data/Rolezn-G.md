## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract` | 5 | 250 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Avoid repeating computations | 4 | - |
| [GAS&#x2011;3](#GAS&#x2011;3) | Consider using solady's 'FixedPointMathLib' | 63 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Counting down in `for` statements is more gas efficient | 12 | 3084 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 13 | 364 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `delete` statement can save gas | 6 | 48 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Increments can be `unchecked` to save gas | 27 | 810 |
| [GAS&#x2011;8](#GAS&#x2011;8) | The result of a function call should be cached rather than re-calling the function | 26 | 1300 |
| [GAS&#x2011;9](#GAS&#x2011;9) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 2 | 12 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Use assembly to validate `msg.sender` | 2 | 24 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Using XOR (^) and AND (&) bitwise equivalents for gas optimizations | 117 | 1521 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Using assembly to check for zero can save gas | 39 | 234 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Using `constant`s instead of `enum` can save gas | 1 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | The more likely conditional `require` can be checked first | 3 | - |
| [GAS&#x2011;15](#GAS&#x2011;15) | Move `proposals.ballotForID(ballotID)` call after the conditional checks to save gas | 1 | - |
| [GAS&#x2011;16](#GAS&#x2011;16) | Move `uint256 minUnstakePercent = stakingConfig.minUnstakePercent();` call after the conditional checks to save gas | 1 | - |

Total: 379 contexts over 16 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract`

#### <ins>Proof Of Concept</ins>


<details>


```solidity
File: DAO.sol

170: if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L170

```solidity
File: DAO.sol

246: uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L246

```solidity
File: DAO.sol

300: uint256 depositedWETH = pools.depositedUserBalance(address(this), weth );

307: withdrawnAmount = weth.balanceOf( address(this) );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L300

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L307



```solidity
File: DAO.sol

365: uint256 liquidityHeld = collateralAndLiquidity.userShareForPool( address(this), poolID );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L365


</details>



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Avoid repeating computations

The following instances show only the first instance of the repeated computations.

#### <ins>Proof Of Concept</ins>


<details>



```solidity
File: DAO.sol

//@audit bootstrappingRewards * 2 can be cached

247: require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
265: exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L247-L265

```solidity
File: PoolMath.sol

//@audit zapAmountA * reserveB can be cached
//@audit reserveA * zapAmountB can be cached

215: if ( zapAmountA * reserveB > reserveA * zapAmountB )
219: if ( zapAmountA * reserveB < reserveA * zapAmountB )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L215-L219



</details>



### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Consider using solady's 'FixedPointMathLib'

Using Solady's 'FixedPointMathLib' for multiplication or division operations in Solidity can lead to significant gas savings. This library is designed to optimize fixed-point arithmetic operations, which are common in financial calculations involving tokens or currencies. By implementing more efficient algorithms and assembly optimizations, 'FixedPointMathLib' minimizes the computational resources required for these operations. 

For contracts that frequently perform such calculations, integrating this library can reduce transaction costs, thereby enhancing overall performance and cost-effectiveness. However, developers must ensure compatibility with their existing codebase and thoroughly test for accuracy and expected behavior to avoid any unintended consequences.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: Upkeep.sol

119: uint256 rewardAmount = withdrawnAmount * daoConfig.upkeepRewardPercent() / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L119

```solidity
File: Upkeep.sol

152: uint256 amountOfWETH = wethBalance * stableConfig.percentArbitrageProfitsForStablePOL() / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L152

```solidity
File: Upkeep.sol

165: uint256 amountOfWETH = wethBalance * daoConfig.arbitrageProfitsPercentPOL() / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L165

```solidity
File: Upkeep.sol

289: return daoWETH * daoConfig.upkeepRewardPercent() / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L289

```solidity
File: ArbitrageSearch.sol

68: uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);

69: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);

70: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L68-L70

```solidity
File: ArbitrageSearch.sol

129: uint256 amountOut = (reservesA1 * bestArbAmountIn) / (reservesA0 + bestArbAmountIn);

130: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);

131: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L129-L131

```solidity
File: DAO.sol

348: uint256 saltToBurn = ( remainingSALT * daoConfig.percentPolRewardsBurned() ) / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L348

```solidity
File: DAO.sol

369: uint256 liquidityToWithdraw = (liquidityHeld * percentToLiquidate) / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L369

```solidity
File: Proposals.sol

90: uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L90

```solidity
File: Proposals.sol

202: uint256 maxSendable = balance * 5 / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L202

```solidity
File: Proposals.sol

324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

335: uint256 minimumQuorum = totalSupply * 5 / 1000;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L324

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L326

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L329

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L335



```solidity
File: PoolMath.sol

192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );

193: uint256 discriminant = B * B + 4 * A * C;

203: swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L192

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L193

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L203



```solidity
File: PoolMath.sol

215: if ( zapAmountA * reserveB > reserveA * zapAmountB )

219: if ( zapAmountA * reserveB < reserveA * zapAmountB )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L215

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L219



```solidity
File: Pools.sol

109: uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;

115: addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;

132: addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;

134: addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L109

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L115

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L132

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L134



```solidity
File: Pools.sol

179: reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;

180: reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L179-L180

```solidity
File: Pools.sol

260: amountOut = reserve0 * amountIn / reserve1;

266: amountOut = reserve1 * amountIn / reserve0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L260

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L266



```solidity
File: Pools.sol

313: swapAmountInValueInETH = ( swapAmountIn * reservesWETH ) / reservesTokenIn;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L313

```solidity
File: PoolUtils.sol

61: uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L61

```solidity
File: CoreSaltyFeed.sol

40: return ( reservesUSDS * 10**8 ) / reservesWBTC;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreSaltyFeed.sol#L40

```solidity
File: CoreSaltyFeed.sol

52: return ( reservesUSDS * 10**18 ) / reservesWETH;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreSaltyFeed.sol#L52

```solidity
File: CoreUniswapFeed.sol

70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );

72: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L70

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L72



```solidity
File: CoreUniswapFeed.sol

112: return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L112

```solidity
File: PriceAggregator.sol

142: if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L142

```solidity
File: Emissions.sol

55: uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L55

```solidity
File: RewardsEmitter.sol

121: uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L121

```solidity
File: SaltRewards.sol

67: uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L67

```solidity
File: SaltRewards.sol

139: uint256 directRewardsForSaltUSDS = ( saltRewardsToDistribute * rewardsConfig.percentRewardsSaltUSDS() ) / 100;

143: uint256 stakingRewardsAmount = ( remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L139

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L143



```solidity
File: CollateralAndLiquidity.sol

159: uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;

160: uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;

167: rewardedWBTC = (rewardedWBTC * maxRewardValue) / rewardValue;

168: rewardedWETH = (rewardedWETH * maxRewardValue) / rewardValue;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L159

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L160

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L167

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L168



```solidity
File: CollateralAndLiquidity.sol

202: uint256 btcValue = ( amountBTC * btcPrice ) / wbtcTenToTheDecimals;

203: uint256 ethValue = ( amountETH * ethPrice ) / wethTenToTheDecimals;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L202-L203

```solidity
File: CollateralAndLiquidity.sol

232: uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares;

233: uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L232-L233

```solidity
File: CollateralAndLiquidity.sol

250: uint256 requiredCollateralValueAfterWithdrawal = ( usdsBorrowedByUsers[wallet] * stableConfig.initialCollateralRatioPercent() ) / 100;

261: return userCollateralAmount * maxWithdrawableValue / userCollateralValue;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L250

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L261



```solidity
File: CollateralAndLiquidity.sol

280: uint256 maxBorrowableAmount = ( userCollateralValue * 100 ) / stableConfig.initialCollateralRatioPercent();


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L280

```solidity
File: CollateralAndLiquidity.sol

307: return (( userCollateralValue * 100 ) / usdsBorrowedAmount) < stableConfig.minimumCollateralRatioPercent();

326: uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;

329: uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L307

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L326

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L329



```solidity
File: Staking.sol

210: uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );

211: return numerator / ( 100 * unstakeRange );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L210-L211

```solidity
File: StakingRewards.sol

114: uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];

118: uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L114

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L118



```solidity
File: StakingRewards.sol

243: uint256 rewardsShare = ( totalRewards[poolID] * user.userShare ) / totalShares[poolID];


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L243



</details>



### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: ArbitrageSearch.sol

115: for( uint256 i = 0; i < 8; i++ )
				{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L115

```solidity
File: Proposals.sol

423: for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L423

```solidity
File: PoolStats.sol

65: for( uint256 i = 0; i < poolIDs.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L65

```solidity
File: PoolStats.sol

81: for( uint256 i = 0; i < poolIDs.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L81

```solidity
File: PoolStats.sol

106: for( uint256 i = 0; i < poolIDs.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L106

```solidity
File: RewardsEmitter.sol

60: for( uint256 i = 0; i < addedRewards.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L60

```solidity
File: RewardsEmitter.sol

116: for( uint256 i = 0; i < poolIDs.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L116

```solidity
File: SaltRewards.sol

64: for( uint256 i = 0; i < addedRewards.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L64

```solidity
File: CollateralAndLiquidity.sol

321: for ( uint256 i = startIndex; i <= endIndex; i++ )
				{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L321

```solidity
File: StakingRewards.sol

152: for( uint256 i = 0; i < poolIDs.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L152

```solidity
File: StakingRewards.sol

185: for( uint256 i = 0; i < addedRewards.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L185

```solidity
File: StakingRewards.sol

296: for( uint256 i = 0; i < cooldowns.length; i++ )
			{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L296



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |



### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Proposals.sol

224: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

233: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L224

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L233



```solidity
File: SaltRewards.sol

60: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

120: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L60

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L120



```solidity
File: CollateralAndLiquidity.sol

83: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

98: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

117: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L83

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L98

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L117



```solidity
File: Staking.sol

90: require( msg.sender == u.wallet, "Sender is not the original staker" );

106: require( msg.sender == u.wallet, "Sender is not the original staker" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L90

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L106



```solidity
File: StakingRewards.sol

59: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );

190: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );

67: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

107: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L59

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L190

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L67

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L107





</details>




### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

<details>


```solidity
File: PoolStats.sol

54: _arbitrageProfits[ poolIDs[i] ] = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L54


```solidity
File: CoreChainlinkFeed.sol

30: price = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreChainlinkFeed.sol#L30


```solidity
File: CollateralAndLiquidity.sol

184: usdsBorrowedByUsers[wallet] = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L184


```solidity
File: Liquidizer.sol

113: usdsThatShouldBeBurned = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L113

```solidity
File: StakingRewards.sol

151: claimableRewards = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L151

```solidity
File: StakingRewards.sol

301: cooldowns[i] = 0;

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L301



</details>



### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Increments can be `unchecked` to save gas

Using `unchecked` increments can save gas by bypassing the built-in overflow checks. This can save [30-40 gas](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) **per iteration**. So it is recommended to use `unchecked` increments when overflow is not possible.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Proposals.sol

108: ballotID = nextBallotID++;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L108

```solidity
File: Proposals.sol

423: for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L423

```solidity
File: BootstrapBallot.sol

57: startExchangeYes++;

59: startExchangeNo++;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/BootstrapBallot.sol#L57

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/BootstrapBallot.sol#L59



```solidity
File: PoolStats.sol

53: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L53

```solidity
File: PoolStats.sol

65: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L65

```solidity
File: PoolStats.sol

81: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L81

```solidity
File: PoolStats.sol

106: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L106

```solidity
File: PriceAggregator.sol

113: numNonZero++;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L113

```solidity
File: RewardsEmitter.sol

60: for( uint256 i = 0; i < addedRewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L60

```solidity
File: RewardsEmitter.sol

116: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L116

```solidity
File: RewardsEmitter.sol

146: for( uint256 i = 0; i < rewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L146

```solidity
File: SaltRewards.sol

64: for( uint256 i = 0; i < addedRewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L64

```solidity
File: SaltRewards.sol

129: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L129

```solidity
File: CollateralAndLiquidity.sol

321: for ( uint256 i = startIndex; i <= endIndex; i++ )

333: liquidatableUsers[count++] = wallet;

338: for ( uint256 i = 0; i < count; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L321

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L333

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L338



```solidity
File: Staking.sol

67: unstakeID = nextUnstakeID++;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L67

```solidity
File: Staking.sol

163: for(uint256 i = start; i <= end; i++)

164: unstakes[index++] = _unstakesByID[ userUnstakes[i]];


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L163-L164

```solidity
File: StakingRewards.sol

152: for( uint256 i = 0; i < poolIDs.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L152

```solidity
File: StakingRewards.sol

185: for( uint256 i = 0; i < addedRewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L185

```solidity
File: StakingRewards.sol

216: for( uint256 i = 0; i < shares.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L216

```solidity
File: StakingRewards.sol

226: for( uint256 i = 0; i < rewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L226

```solidity
File: StakingRewards.sol

260: for( uint256 i = 0; i < rewards.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L260

```solidity
File: StakingRewards.sol

277: for( uint256 i = 0; i < shares.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L277

```solidity
File: StakingRewards.sol

296: for( uint256 i = 0; i < cooldowns.length; i++ )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L296



</details>



### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: Upkeep.sol

//@audit payable(exchangeConfig.teamVestingWallet(), instead use the immutable variable

233: uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));

236: VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L233

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L236



```solidity
File: DAO.sol

//@audit exchangeConfig.salt(), instead use the immutable variable

170: if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )

172: IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );

189: emit GeoExclusionUpdated(ballot.string1, false, exchangeConfig.accessManager().geoVersion());

197: exchangeConfig.accessManager().excludedCountriesUpdated();

199: emit GeoExclusionUpdated(ballot.string1, true, exchangeConfig.accessManager().geoVersion());


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L170

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L172

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L189

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L197

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L199



```solidity
File: DAO.sol

//@audit exchangeConfig calls, instead use the immutable variable

246: uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );

254: poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );

255: poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

257: bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );

258: bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );

265: exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L246

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L254

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L255

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L257

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L258

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L265



```solidity
File: DAO.sol

//@audit exchangeConfig.upkeep(), instead use the immutable variable

318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L318

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L329



```solidity
File: Proposals.sol

//@audit exchangeConfig calls, instead use the immutable variable

124: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

169: require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );

182: require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );

183: require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );

184: require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L124

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L132

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L169

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L182

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L183

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L184



```solidity
File: Proposals.sol

//@audit daoConfig.baseBallotQuorumPercentTimes1000()

324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L324

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L326

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L329



```solidity
File: Airdrop.sol

//@audit numberAuthorized()

60: require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

64: saltAmountForEachUser = saltBalance / numberAuthorized();


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/Airdrop.sol#L60

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/Airdrop.sol#L64



</details>



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.23/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
File: Pools.sol

35: uint128 reserve0;
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L35

```solidity
File: Pools.sol

36: uint128 reserve1;
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L36



### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender with the least amount of opcodes necessary. 

#### <ins>Proof Of Concept</ins>

```solidity
File: DAO.sol

362: require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L362

```solidity
File: PoolStats.sol

49: require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L49



### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Using XOR (^) and AND (&) bitwise equivalents for gas optimizations

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: AccessManager.sol

43: require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/AccessManager.sol#L43

```solidity
File: ExchangeConfig.sol

77: if ( wallet == address(dao) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ExchangeConfig.sol#L77

```solidity
File: ExchangeConfig.sol

81: if ( wallet == address(airdrop) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ExchangeConfig.sol#L81

```solidity
File: ManagedWallet.sol

44: require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L44

```solidity
File: ManagedWallet.sol

49: require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L49

```solidity
File: ManagedWallet.sol

61: require( msg.sender == confirmationWallet, "Invalid sender" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L61

```solidity
File: ManagedWallet.sol

76: require( msg.sender == proposedMainWallet, "Invalid sender" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L76

```solidity
File: SigningTools.sol

13: require( signature.length == 65, "Invalid signature length" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/SigningTools.sol#L13

```solidity
File: SigningTools.sol

28: return (recoveredAddress == EXPECTED_SIGNER);

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/SigningTools.sol#L28

```solidity
File: Upkeep.sol

115: if ( withdrawnAmount == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L115

```solidity
File: Upkeep.sol

148: if ( wethBalance == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L148

```solidity
File: Upkeep.sol

161: if ( wethBalance == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L161

```solidity
File: Upkeep.sol

174: if ( wethBalance == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L174

```solidity
File: DAO.sol

119: if ( winningVote == Vote.INCREASE )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L119

```solidity
File: DAO.sol

121: else if ( winningVote == Vote.DECREASE )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L121

```solidity
File: DAO.sol

135: if ( nameHash == keccak256(bytes( "setContract:priceFeed1_confirm" )) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L135

```solidity
File: DAO.sol

137: else if ( nameHash == keccak256(bytes( "setContract:priceFeed2_confirm" )) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L137

```solidity
File: DAO.sol

139: else if ( nameHash == keccak256(bytes( "setContract:priceFeed3_confirm" )) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L139

```solidity
File: DAO.sol

141: else if ( nameHash == keccak256(bytes( "setContract:accessManager_confirm" )) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L141

```solidity
File: DAO.sol

157: if ( ballot.ballotType == BallotType.UNWHITELIST_TOKEN )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L157

```solidity
File: DAO.sol

166: else if ( ballot.ballotType == BallotType.SEND_SALT )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L166

```solidity
File: DAO.sol

178: else if ( ballot.ballotType == BallotType.CALL_CONTRACT )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L178

```solidity
File: DAO.sol

185: else if ( ballot.ballotType == BallotType.INCLUDE_COUNTRY )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L185

```solidity
File: DAO.sol

192: else if ( ballot.ballotType == BallotType.EXCLUDE_COUNTRY )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L192

```solidity
File: DAO.sol

203: else if ( ballot.ballotType == BallotType.SET_CONTRACT )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L203

```solidity
File: DAO.sol

207: else if ( ballot.ballotType == BallotType.SET_WEBSITE_URL )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L207

```solidity
File: DAO.sol

210: else if ( ballot.ballotType == BallotType.CONFIRM_SET_CONTRACT )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L210

```solidity
File: DAO.sol

213: else if ( ballot.ballotType == BallotType.CONFIRM_SET_WEBSITE_URL )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L213

```solidity
File: DAO.sol

251: require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L251

```solidity
File: DAO.sol

285: if ( ballot.ballotType == BallotType.PARAMETER )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L285

```solidity
File: DAO.sol

287: else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L287

```solidity
File: DAO.sol

297: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L297

```solidity
File: DAO.sol

301: if ( depositedWETH == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L301

```solidity
File: DAO.sol

318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L318

```solidity
File: DAO.sol

329: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L329

```solidity
File: DAO.sol

337: if ( claimedSALT == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L337

```solidity
File: DAO.sol

362: require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L362

```solidity
File: DAO.sol

366: if ( liquidityHeld == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L366

```solidity
File: Parameters.sol

60: if ( parameterType == ParameterTypes.maximumWhitelistedPools )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L60

```solidity
File: Parameters.sol

62: else if ( parameterType == ParameterTypes.maximumInternalSwapPercentTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L62

```solidity
File: Parameters.sol

66: else if ( parameterType == ParameterTypes.minUnstakeWeeks )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L66

```solidity
File: Parameters.sol

68: else if ( parameterType == ParameterTypes.maxUnstakeWeeks )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L68

```solidity
File: Parameters.sol

70: else if ( parameterType == ParameterTypes.minUnstakePercent )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L70

```solidity
File: Parameters.sol

72: else if ( parameterType == ParameterTypes.modificationCooldown )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L72

```solidity
File: Parameters.sol

76: else if ( parameterType == ParameterTypes.rewardsEmitterDailyPercentTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L76

```solidity
File: Parameters.sol

78: else if ( parameterType == ParameterTypes.emissionsWeeklyPercentTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L78

```solidity
File: Parameters.sol

80: else if ( parameterType == ParameterTypes.stakingRewardsPercent )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L80

```solidity
File: Parameters.sol

82: else if ( parameterType == ParameterTypes.percentRewardsSaltUSDS )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L82

```solidity
File: Parameters.sol

86: else if ( parameterType == ParameterTypes.rewardPercentForCallingLiquidation )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L86

```solidity
File: Parameters.sol

88: else if ( parameterType == ParameterTypes.maxRewardValueForCallingLiquidation )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L88

```solidity
File: Parameters.sol

90: else if ( parameterType == ParameterTypes.minimumCollateralValueForBorrowing )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L90

```solidity
File: Parameters.sol

92: else if ( parameterType == ParameterTypes.initialCollateralRatioPercent )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L92

```solidity
File: Parameters.sol

94: else if ( parameterType == ParameterTypes.minimumCollateralRatioPercent )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L94

```solidity
File: Parameters.sol

96: else if ( parameterType == ParameterTypes.percentArbitrageProfitsForStablePOL )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L96

```solidity
File: Parameters.sol

100: else if ( parameterType == ParameterTypes.bootstrappingRewards )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L100

```solidity
File: Parameters.sol

102: else if ( parameterType == ParameterTypes.percentPolRewardsBurned )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L102

```solidity
File: Parameters.sol

104: else if ( parameterType == ParameterTypes.baseBallotQuorumPercentTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L104

```solidity
File: Parameters.sol

106: else if ( parameterType == ParameterTypes.ballotDuration )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L106

```solidity
File: Parameters.sol

108: else if ( parameterType == ParameterTypes.requiredProposalPercentStakeTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L108

```solidity
File: Parameters.sol

110: else if ( parameterType == ParameterTypes.maxPendingTokensForWhitelisting )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L110

```solidity
File: Parameters.sol

112: else if ( parameterType == ParameterTypes.arbitrageProfitsPercentPOL )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L112

```solidity
File: Parameters.sol

114: else if ( parameterType == ParameterTypes.upkeepRewardPercent )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L114

```solidity
File: Parameters.sol

118: else if ( parameterType == ParameterTypes.maximumPriceFeedPercentDifferenceTimes1000 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L118

```solidity
File: Parameters.sol

120: else if ( parameterType == ParameterTypes.setPriceFeedCooldown )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L120

```solidity
File: Proposals.sol

124: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L124

```solidity
File: Proposals.sol

132: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L132

```solidity
File: Proposals.sol

137: if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L137

```solidity
File: Proposals.sol

224: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L224

```solidity
File: Proposals.sol

267: if ( ballot.ballotType == BallotType.PARAMETER )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L267

```solidity
File: Proposals.sol

268: require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L268

```solidity
File: Proposals.sol

270: require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L270

```solidity
File: Proposals.sol

323: if ( ballotType == BallotType.PARAMETER )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L323

```solidity
File: Proposals.sol

325: else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L325

```solidity
File: Proposals.sol

347: if ( ballot.ballotType == BallotType.PARAMETER )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L347

```solidity
File: Airdrop.sol

48: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/Airdrop.sol#L48

```solidity
File: Airdrop.sol

58: require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/Airdrop.sol#L58

```solidity
File: InitialDistribution.sol

52: require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/InitialDistribution.sol#L52

```solidity
File: Pools.sol

72: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L72

```solidity
File: Pools.sol

97: if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L97

```solidity
File: Pools.sol

142: require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L142

```solidity
File: Pools.sol

172: require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L172

```solidity
File: Pools.sol

325: if ( swapAmountInValueInETH == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L325

```solidity
File: PoolsConfig.sol

122: return ( poolID == PoolUtils.STAKED_SALT ) || _whitelist.contains( poolID );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolsConfig.sol#L122

```solidity
File: PoolStats.sol

49: require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L49

```solidity
File: PoolStats.sol

67: if (poolID == poolIDs[i])

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L67

```solidity
File: PoolUtils.sol

56: if ( amountIn == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L56

```solidity
File: CoreUniswapFeed.sol

103: if ((uniswapWBTC_WETH == 0) || (uniswapWETH_USDC == 0 ))

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L103

```solidity
File: CoreUniswapFeed.sol

122: if ( uniswapWETH_USDC == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L122

```solidity
File: PriceAggregator.sol

53: if ( priceFeedNum == 1 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L53

```solidity
File: PriceAggregator.sol

55: else if ( priceFeedNum == 2 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L55

```solidity
File: PriceAggregator.sol

57: else if ( priceFeedNum == 3 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L57

```solidity
File: Emissions.sol

42: require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L42

```solidity
File: Emissions.sol

44: if ( timeSinceLastUpkeep == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L44

```solidity
File: Emissions.sol

56: if ( saltToSend == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L56

```solidity
File: RewardsEmitter.sol

84: require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L84

```solidity
File: RewardsEmitter.sol

86: if ( timeSinceLastUpkeep == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L86

```solidity
File: SaltRewards.sol

60: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L60

```solidity
File: SaltRewards.sol

70: if ( poolID == saltUSDSPoolID )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L70

```solidity
File: SaltRewards.sol

108: require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L108

```solidity
File: SaltRewards.sol

119: require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L119

```solidity
File: SaltRewards.sol

120: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L120

```solidity
File: SaltRewards.sol

124: if ( saltRewardsToDistribute == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L124

```solidity
File: SaltRewards.sol

134: if ( totalProfits == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L134

```solidity
File: CollateralAndLiquidity.sol

224: if ( userCollateralAmount == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L224

```solidity
File: CollateralAndLiquidity.sol

246: if ( userCollateralAmount == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L246

```solidity
File: CollateralAndLiquidity.sol

301: if ( usdsBorrowedAmount == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L301

```solidity
File: Liquidizer.sol

83: require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L83

```solidity
File: Liquidizer.sol

104: if ( usdsThatShouldBeBurned == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L104

```solidity
File: Liquidizer.sol

134: require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L134

```solidity
File: USDS.sol

42: require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/USDS.sol#L42

```solidity
File: Staking.sol

88: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L88

```solidity
File: Staking.sol

90: require( msg.sender == u.wallet, "Sender is not the original staker" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L90

```solidity
File: Staking.sol

104: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L104

```solidity
File: Staking.sol

106: require( msg.sender == u.wallet, "Sender is not the original staker" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L106

```solidity
File: Staking.sol

132: require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L132

```solidity
File: Staking.sol

175: if ( unstakeIDs.length == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L175

```solidity
File: StakingRewards.sol

239: if ( user.userShare == 0 )

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L239



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |





### <a href="#gas-summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: Upkeep.sol

115: if ( withdrawnAmount == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L115

```solidity
File: Upkeep.sol

148: if ( wethBalance == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L148

```solidity
File: Upkeep.sol

161: if ( wethBalance == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L161

```solidity
File: Upkeep.sol

174: if ( wethBalance == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Upkeep.sol#L174

```solidity
File: DAO.sol

301: if ( depositedWETH == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L301

```solidity
File: DAO.sol

337: if ( claimedSALT == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L337

```solidity
File: DAO.sol

366: if ( liquidityHeld == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L366

```solidity
File: Proposals.sol

102: require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );

103: require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L102-L103

```solidity
File: Proposals.sol

321: require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L321

```solidity
File: Pools.sol

97: if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L97

```solidity
File: Pools.sol

325: if ( swapAmountInValueInETH == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L325

```solidity
File: PoolUtils.sol

56: if ( amountIn == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L56

```solidity
File: CoreUniswapFeed.sol

103: if ((uniswapWBTC_WETH == 0) || (uniswapWETH_USDC == 0 ))


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L103

```solidity
File: CoreUniswapFeed.sol

122: if ( uniswapWETH_USDC == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreUniswapFeed.sol#L122

```solidity
File: PriceAggregator.sol

183: require (price != 0, "Invalid BTC price" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L183

```solidity
File: PriceAggregator.sol

195: require (price != 0, "Invalid ETH price" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/PriceAggregator.sol#L195

```solidity
File: Emissions.sol

44: if ( timeSinceLastUpkeep == 0 )

56: if ( saltToSend == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L44

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/Emissions.sol#L56



```solidity
File: RewardsEmitter.sol

66: if ( amountToAdd != 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L66

```solidity
File: RewardsEmitter.sol

86: if ( timeSinceLastUpkeep == 0 )

124: if ( amountToAddForPool != 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L86

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/RewardsEmitter.sol#L124



```solidity
File: SaltRewards.sol

124: if ( saltRewardsToDistribute == 0 )

134: if ( totalProfits == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L124

https://github.com/code-423n4/2024-01-salty/tree/main/src/rewards/SaltRewards.sol#L134



```solidity
File: CollateralAndLiquidity.sol

131: if ( usdsBorrowedByUsers[msg.sender] == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L131

```solidity
File: CollateralAndLiquidity.sol

224: if ( userCollateralAmount == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L224

```solidity
File: CollateralAndLiquidity.sol

246: if ( userCollateralAmount == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L246

```solidity
File: CollateralAndLiquidity.sol

271: if ( userShareForPool( wallet, collateralPoolID ) == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L271

```solidity
File: CollateralAndLiquidity.sol

301: if ( usdsBorrowedAmount == 0 )

320: if ( totalCollateralValue != 0 )

347: if ( numberOfUsersWithBorrowedUSDS() == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L301

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L320

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L347



```solidity
File: Liquidizer.sol

104: if ( usdsThatShouldBeBurned == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L104

```solidity
File: Staking.sol

175: if ( unstakeIDs.length == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L175

```solidity
File: StakingRewards.sol

60: require( increaseShareAmount != 0, "Cannot increase zero share" );

78: if ( existingTotalShares != 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L60

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L78



```solidity
File: StakingRewards.sol

99: require( decreaseShareAmount != 0, "Cannot decrease zero share" );

136: if ( claimableRewards != 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L99

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L136



```solidity
File: StakingRewards.sol

235: if ( totalShares[poolID] == 0 )

239: if ( user.userShare == 0 )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L235

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L239





</details>



### <a href="#gas-summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Using `constant`s instead of `enum` can save gas

`Enum` is expensive and it is [more efficient to use constants](https://www.codehawks.com/finding/clm84992q02j9w9ruebun36d9) instead. An illustrative example of this approach can be found in the [ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/181d518609a9f006fcb97af63e6952e603cf100e/contracts/utils/ReentrancyGuard.sol#L34-L35).

#### <ins>Proof Of Concept</ins>


```solidity
File: Parameters.sol

14: enum ParameterTypes {

		// PoolsConfig
		maximumWhitelistedPools,
		maximumInternalSwapPercentTimes1000,

		// StakingConfig
		minUnstakeWeeks,
		maxUnstakeWeeks,
		minUnstakePercent,
		modificationCooldown,

		// RewardsConfig
    	rewardsEmitterDailyPercentTimes1000,
		emissionsWeeklyPercentTimes1000,
		stakingRewardsPercent,
		percentRewardsSaltUSDS,

		// StableConfig
		rewardPercentForCallingLiquidation,
		maxRewardValueForCallingLiquidation,
		minimumCollateralValueForBorrowing,
		initialCollateralRatioPercent,
		minimumCollateralRatioPercent,
		percentArbitrageProfitsForStablePOL,

		// DAOConfig
		bootstrappingRewards,
		percentPolRewardsBurned,
		baseBallotQuorumPercentTimes1000,
		ballotDuration,
		requiredProposalPercentStakeTimes1000,
		maxPendingTokensForWhitelisting,
		arbitrageProfitsPercentPOL,
		upkeepRewardPercent,

		// PriceAggregator
		maximumPriceFeedPercentDifferenceTimes1000,
		setPriceFeedCooldown
		}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Parameters.sol#L14


### <a href="#gas-summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> The more likely conditional `require` can be checked first

Conditional checks that are more likely to happen should be checked first. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

#### <ins>Proof Of Concept</ins>

1: The caller of `changeWallets` will most likely will be called by `proposedMainWallet`, which makes it wasteful to check two conditional `require`.
`require( block.timestamp >= activeTimelock, "Timelock not yet completed" );` would be the more relevant check.

```solidity
File: ManagedWallet.sol

function changeWallets() external
		{
		
		require( msg.sender == proposedMainWallet, "Invalid sender" );
		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

		
		mainWallet = proposedMainWallet;
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet);

		
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}
	}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L77

#### Optimized Code

```solidity
File: ManagedWallet.sol

function changeWallets() external
		{
			
		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
		require( msg.sender == proposedMainWallet, "Invalid sender" );
		
		mainWallet = proposedMainWallet;
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet);

		
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}
	}
```

2: The caller of `distributionApproved` will most likely will be called by `bootstrapBallot`, which makes it wasteful to check two conditional `require`.
`require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );` would be the more relevant check.


```solidity
File: InitialDistribution.sol

function distributionApproved() external
    	{
    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );

    	
		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );

	    
		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );

	    
		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );

	    
		salt.safeTransfer( address(airdrop), 5 * MILLION_ETHER );
		airdrop.allowClaiming();

		bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

	    
	    
		salt.safeTransfer( address(saltRewards), 8 * MILLION_ETHER );
		saltRewards.sendInitialSaltRewards(5 * MILLION_ETHER, poolIDs );
    	}
	}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/InitialDistribution.sol#L53

#### Optimized Code

```solidity
File: InitialDistribution.sol

function distributionApproved() external
    	{
		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
		require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );

    	
		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );

	    
		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );

	    
		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );

	    
		salt.safeTransfer( address(airdrop), 5 * MILLION_ETHER );
		airdrop.allowClaiming();

		bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

	    
	    
		salt.safeTransfer( address(saltRewards), 8 * MILLION_ETHER );
		saltRewards.sendInitialSaltRewards(5 * MILLION_ETHER, poolIDs );
    	}
	}
```

3: The caller of `mintTo` will most likely will be called by `collateralAndLiquidity`, which makes it wasteful to check two conditional `require`.
`require( amount > 0, "Cannot mint zero USDS" );` would be the more relevant check.


```solidity
File: USDS.sol

function mintTo( address wallet, uint256 amount ) external
		{
		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
		require( amount > 0, "Cannot mint zero USDS" );

		_mint( wallet, amount );

		emit USDSMinted(wallet, amount);
		}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/USDS.sol#L43

#### Optimized Code

```solidity
File: USDS.sol

function mintTo( address wallet, uint256 amount ) external
		{
		require( amount > 0, "Cannot mint zero USDS" );
		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );

		_mint( wallet, amount );

		emit USDSMinted(wallet, amount);
		}
```


### <a href="#gas-summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Move `proposals.ballotForID(ballotID)` call after the conditional checks to save gas

In `_finalizeTokenWhitelisting()`, the call for `Ballot memory ballot = proposals.ballotForID(ballotID);` is only used after two `require` checks. Gas can be saved by moving the `ballot` variable after the `require` checks.

#### <ins>Proof Of Concept</ins>


```solidity
File: DAO.sol

function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
			
			Ballot memory ballot = proposals.ballotForID(ballotID);

			uint256 bootstrappingRewards = daoConfig.bootstrappingRewards();

			
			
			uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

			
			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

			
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );

			
			AddedReward[] memory addedRewards = new AddedReward[](2);
			addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
			addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
			liquidityRewardsEmitter.addSALTRewards( addedRewards );

			emit WhitelistToken(IERC20(ballot.address1));
			}

		
		proposals.markBallotAsFinalized(ballotID);
		}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L251


#### Optimized Code

```solidity
File: DAO.sol

function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{

			uint256 bootstrappingRewards = daoConfig.bootstrappingRewards();

			
			
			uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

			
			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

			Ballot memory ballot = proposals.ballotForID(ballotID);
			
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );

			
			AddedReward[] memory addedRewards = new AddedReward[](2);
			addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
			addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
			liquidityRewardsEmitter.addSALTRewards( addedRewards );

			emit WhitelistToken(IERC20(ballot.address1));
			}

		
		proposals.markBallotAsFinalized(ballotID);
		}
```


### <a href="#gas-summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Move `uint256 minUnstakePercent = stakingConfig.minUnstakePercent();` call after the conditional checks to save gas

In `calculateUnstake()`, the call for `uint256 minUnstakePercent = stakingConfig.minUnstakePercent();` is only used after two `require` checks. Gas can be saved by moving the `minUnstakePercent` variable after the `require` checks.

#### <ins>Proof Of Concept</ins>

```solidity
File: Staking.sol

function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
		{
		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );

		uint256 percentAboveMinimum = 100 - minUnstakePercent;
		uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;

		uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
    	return numerator / ( 100 * unstakeRange );
		}
	}
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L204-L205


#### Optimized Code

```solidity
File: Staking.sol

function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
		{
		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
        
		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );

		uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

		uint256 percentAboveMinimum = 100 - minUnstakePercent;
		uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;

		uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
    	return numerator / ( 100 * unstakeRange );
		}
	}
```