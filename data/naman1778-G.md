## [G-01] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 6 instances of this issue in 4 files:

```
File: Proposals.sol	

268: require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );

270: require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );

325: else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
``` 
    diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
    index 40ebd17..6958891 100644
    --- a/src/dao/Proposals.sol
    +++ b/src/dao/Proposals.sol
    @@ -265,9 +265,9 @@ contract Proposals is IProposals, ReentrancyGuard

                    // Make sure that the vote type is valid for the given ballot
                    if ( ballot.ballotType == BallotType.PARAMETER )
    -                       require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
    +                       require( ((vote ^ Vote.INCREASE) & (vote ^ Vote.DECREASE) & (vote ^ Vote.NO_CHANGE)) == 0, "Invalid VoteType for Parameter Ballot" );
                    else // If a Ballot is not a Parameter Ballot, it is an Approval ballot
    -                       require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
    +                       require( ((vote ^ Vote.YES) & (vote ^ Vote.NO)) == 0, "Invalid VoteType for Approval Ballot" );

                    // Make sure that the user has voting power before proceeding.
                    // Voting power is equal to their userShare of STAKED_SALT.
    @@ -322,7 +322,7 @@ contract Proposals is IProposals, ReentrancyGuard

                    if ( ballotType == BallotType.PARAMETER )
                            requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
    -               else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
    +               else if ( (( ballotType ^ BallotType.WHITELIST_TOKEN ) & ( ballotType ^ BallotType.UNWHITELIST_TOKEN )) == 0 )
                            requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
                    else
                            // All other ballot types require 3x multiple of the baseQuorum
    @@ -443,4 +443,4 @@ contract Proposals is IProposals, ReentrancyGuard
                    {
                    return _userHasActiveProposal[user];
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol

```
File: PoolStats.sol	

95: if ( ( poolIndex1 != indicies.index1 ) || ( poolIndex2 != indicies.index2 ) || ( poolIndex3 != indicies.index3 ) )
```
    diff --git a/src/pools/PoolStats.sol b/src/pools/PoolStats.sol
    index 8e44890..57ef88b 100644
    --- a/src/pools/PoolStats.sol
    +++ b/src/pools/PoolStats.sol
    @@ -92,7 +92,7 @@ abstract contract PoolStats is IPoolStats

                                    // Check if the indicies in storage have the correct values - and if not then update them
                                    ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
    -                               if ( ( poolIndex1 != indicies.index1 ) || ( poolIndex2 != indicies.index2 ) || ( poolIndex3 != indicies.index3 ) )
    +                               if ( (( poolIndex1 ^ indicies.index1 ) | ( poolIndex2 ^ indicies.index2 ) | ( poolIndex3 ^ indicies.index3 )) != 0 )
                                            _arbitrageIndicies[poolID] = ArbitrageIndicies(poolIndex1, poolIndex2, poolIndex3);
                                    }
                            }
    @@ -144,4 +144,4 @@ abstract contract PoolStats is IPoolStats
                    {
                    return _arbitrageIndicies[poolID];
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol

```
File: Pools.sol	
 
97: if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
```
    diff --git a/src/pools/Pools.sol b/src/pools/Pools.sol
    index 7adb563..3042d0b 100644
    --- a/src/pools/Pools.sol
    +++ b/src/pools/Pools.sol
    @@ -94,7 +94,7 @@ contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
                    uint256 reserve1 = reserves.reserve1;

                    // If either reserve is zero then consider the pool to be empty and that the added liquidity will become the initial token ratio
    -               if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
    +               if ( (( reserve0 ^ 0 ) & ( reserve1 ^ 0 )) == 0 )
                            {
                            // Update the reserves
                            reserves.reserve0 += uint128(maxAmount0);
    @@ -424,4 +424,4 @@ contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
                    {
                    return _userDeposits[user][token];
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol

```
File: CoreUniswapFeed.sol	

103: if ((uniswapWBTC_WETH == 0) || (uniswapWETH_USDC == 0 ))
```
    diff --git a/src/price_feed/CoreUniswapFeed.sol b/src/price_feed/CoreUniswapFeed.sol
    index 79c9652..a49c0f0 100644
    --- a/src/price_feed/CoreUniswapFeed.sol
    +++ b/src/price_feed/CoreUniswapFeed.sol
    @@ -100,7 +100,7 @@ contract CoreUniswapFeed is IPriceFeed
             uint256 uniswapWETH_USDC = getUniswapTwapWei( UNISWAP_V3_WETH_USDC, twapInterval );

                    // Return zero if either is invalid
    -        if ((uniswapWBTC_WETH == 0) || (uniswapWETH_USDC == 0 ))
    +        if (((uniswapWBTC_WETH ^ 0) & (uniswapWETH_USDC ^ 0 )) == 0)
                    return 0;

                    if ( wbtc_wethFlipped )
    @@ -142,4 +142,4 @@ contract CoreUniswapFeed is IPriceFeed
                    {
                    return getTwapWETH( TWAP_PERIOD );
                    }
    -    }
    \ No newline at end of file
    +    }

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol

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

#### Gas Report

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

## [G-02] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There is 1 instance of this issue in 1 file:

```
File: CollateralAndLiquidity.sol	

317: uint256 totalCollateralShares = totalShares[collateralPoolID];
```

    diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
    index fc14695..aa0f8f9 100644
    --- a/src/stable/CollateralAndLiquidity.sol
    +++ b/src/stable/CollateralAndLiquidity.sol
    @@ -314,7 +314,6 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                    uint256 count = 0;

                    // Cache
    -               uint256 totalCollateralShares = totalShares[collateralPoolID];
                    uint256 totalCollateralValue = totalCollateralValueInUSD();

                    if ( totalCollateralValue != 0 )
    @@ -326,7 +325,7 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                                    uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;

                                    // Determine minCollateral in terms of minCollateralValue
    -                               uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
    +                               uint256 minCollateral = (minCollateralValue * totalShares[collateralPoolID]) / totalCollateralValue;

                                    // Make sure the user has at least minCollateral
                                    if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-03] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 22 instances of this issue in 8 files:

```
File: ArbitrageSearch.sol	

115: for( uint256 i = 0; i < 8; i++ )
```
    diff --git a/src/arbitrage/ArbitrageSearch.sol b/src/arbitrage/ArbitrageSearch.sol
    index f71030c..7e8f320 100644
    --- a/src/arbitrage/ArbitrageSearch.sol
    +++ b/src/arbitrage/ArbitrageSearch.sol
    @@ -112,7 +112,7 @@ abstract contract ArbitrageSearch
                            uint256 rightPoint = swapAmountInValueInETH + (swapAmountInValueInETH >> 2); // 100% + 25% of swapAmountInValueInETH

                            // Cost is about 492 gas per loop iteration
    -                       for( uint256 i = 0; i < 8; i++ )
    +                       for( uint256 i = 7; i >= 0; i-- )
                                    {
                                    uint256 midpoint = (leftPoint + rightPoint) >> 1;

    @@ -134,4 +134,4 @@ abstract contract ArbitrageSearch
                                    return 0;
                            }
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol

```
File: Proposals.sol	

423: for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
```

    diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
    index 40ebd17..3024237 100644
    --- a/src/dao/Proposals.sol
    +++ b/src/dao/Proposals.sol
    @@ -420,7 +420,7 @@ contract Proposals is IProposals, ReentrancyGuard

                    uint256 bestID = 0;
                    uint256 mostYes = 0;
    -               for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
    +               for( uint256 i = _openBallotsForTokenWhitelisting.length() - 1 ; i >= 0 ; i-- )
                            {
                            uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
                            uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
    @@ -443,4 +443,4 @@ contract Proposals is IProposals, ReentrancyGuard
                    {
                    return _userHasActiveProposal[user];
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol

```
File: PoolStats.sol	

53: for( uint256 i = 0; i < poolIDs.length; i++ )

65: for( uint256 i = 0; i < poolIDs.length; i++ )

81: for( uint256 i = 0; i < poolIDs.length; i++ )

106: for( uint256 i = 0; i < poolIDs.length; i++ )
```

    diff --git a/src/pools/PoolStats.sol b/src/pools/PoolStats.sol
    index 8e44890..d642a21 100644
    --- a/src/pools/PoolStats.sol
    +++ b/src/pools/PoolStats.sol
    @@ -50,7 +50,7 @@ abstract contract PoolStats is IPoolStats

                    bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1; i >= 0 ; i-- )
                            _arbitrageProfits[ poolIDs[i] ] = 0;
                    }

    @@ -62,7 +62,7 @@ abstract contract PoolStats is IPoolStats
                    {
                    bytes32 poolID = PoolUtils._poolID( tokenA, tokenB );

    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1; i >= 0 ; i-- )
                            {
                            if (poolID == poolIDs[i])
                                    return uint64(i);
    @@ -78,7 +78,7 @@ abstract contract PoolStats is IPoolStats
                    {
                    bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1; i >= 0 ; i-- )
                            {
                            bytes32 poolID = poolIDs[i];
                            (IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);
    @@ -103,7 +103,7 @@ abstract contract PoolStats is IPoolStats
            // The calculated sums for each pool will then be used to proportionally distribute SALT rewards to each of the contributing pools.
            function _calculateArbitrageProfits( bytes32[] memory poolIDs, uint256[] memory _calculatedProfits ) internal view
                    {
    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1 ; i >= 0 ; i-- )
                            {
                            // references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
                            bytes32 poolID = poolIDs[i];
    @@ -144,4 +144,4 @@ abstract contract PoolStats is IPoolStats
                    {
                    return _arbitrageIndicies[poolID];
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol

```
File: RewardsEmitter.sol	

60: for( uint256 i = 0; i < addedRewards.length; i++ )

116: for( uint256 i = 0; i < poolIDs.length; i++ )

146: for( uint256 i = 0; i < rewards.length; i++ )
```
    diff --git a/src/rewards/RewardsEmitter.sol b/src/rewards/RewardsEmitter.sol
    index a7f3a47..aa8d733 100644
    --- a/src/rewards/RewardsEmitter.sol
    +++ b/src/rewards/RewardsEmitter.sol
    @@ -57,7 +57,7 @@ contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
            function addSALTRewards( AddedReward[] calldata addedRewards ) external nonReentrant
                    {
                    uint256 sum = 0;
    -               for( uint256 i = 0; i < addedRewards.length; i++ )
    +               for( uint256 i = addedRewards.length - 1; i >= 0 ; i-- )
                            {
                            AddedReward memory addedReward = addedRewards[i];
                            require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );
    @@ -113,7 +113,7 @@ contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
                    uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000

                    uint256 sum = 0;
    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1; i >= 0 ; i-- )
                            {
                            bytes32 poolID = poolIDs[i];

    @@ -143,7 +143,7 @@ contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
                    {
                    uint256[] memory rewards = new uint256[]( pools.length );

    -               for( uint256 i = 0; i < rewards.length; i++ )
    +               for( uint256 i = rewards.length - 1 ; i >= 0 ; i-- )
                            rewards[i] = pendingRewards[ pools[i] ];

                    return rewards;

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol

```
File: SaltRewards.sol	

64: for( uint256 i = 0; i < addedRewards.length; i++ )

87: for( uint256 i = 0; i < addedRewards.length; i++ )

129: for( uint256 i = 0; i < poolIDs.length; i++ )
```
    diff --git a/src/rewards/SaltRewards.sol b/src/rewards/SaltRewards.sol
    index a4994b6..481bccc 100644
    --- a/src/rewards/SaltRewards.sol
    +++ b/src/rewards/SaltRewards.sol
    @@ -61,7 +61,7 @@ contract SaltRewards is ISaltRewards

                    // Send SALT rewards (with an amount of pendingLiquidityRewards) proportional to the profits generated by each pool
                    AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
    -               for( uint256 i = 0; i < addedRewards.length; i++ )
    +               for( uint256 i = addedRewards.length - 1; i >= 0 ; i-- )
                            {
                            bytes32 poolID = poolIDs[i];
                            uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;
    @@ -84,7 +84,7 @@ contract SaltRewards is ISaltRewards
                    uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length; // poolIDs.length is guaranteed to not be zero

                    AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
    -               for( uint256 i = 0; i < addedRewards.length; i++ )
    +               for( uint256 i = addedRewards.length - 1; i >= 0 ; i-- )
                            addedRewards[i] = AddedReward( poolIDs[i], amountPerPool );

                    // Send the liquidity bootstrap rewards to the liquidityRewardsEmitter
    @@ -126,7 +126,7 @@ contract SaltRewards is ISaltRewards

                    // Determine the total profits so we can calculate proportional share for the liquidity rewards
                    uint256 totalProfits = 0;
    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1; i >= 0 ; i-- )
                            totalProfits += profitsForPools[i];

                    // Make sure that there are some profits to determine the proportional liquidity rewards.

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol

```
File: CollateralAndLiquidity.sol	

321: for ( uint256 i = startIndex; i <= endIndex; i++ )

338: for ( uint256 i = 0; i < count; i++ )
```
    diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
    index fc14695..bf244c6 100644
    --- a/src/stable/CollateralAndLiquidity.sol
    +++ b/src/stable/CollateralAndLiquidity.sol
    @@ -318,7 +318,7 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                    uint256 totalCollateralValue = totalCollateralValueInUSD();

                    if ( totalCollateralValue != 0 )
    -                       for ( uint256 i = startIndex; i <= endIndex; i++ )
    +                       for ( uint256 i = endIndex ; i >= startIndex ; i-- )
                                    {
                                    address wallet = _walletsWithBorrowedUSDS.at(i);

    @@ -335,7 +335,7 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity

                    // Resize the array to match the actual number of liquidatable positions found
                    address[] memory resizedLiquidatableUsers = new address[](count);
    -               for ( uint256 i = 0; i < count; i++ )
    +               for ( uint256 i = count - 1 ; i >= 0 ; i-- )
                            resizedLiquidatableUsers[i] = liquidatableUsers[i];

                    return resizedLiquidatableUsers;

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol

```
File: StakingRewards.sol	

152: for( uint256 i = 0; i < poolIDs.length; i++ )

185: for( uint256 i = 0; i < addedRewards.length; i++ )

216: for( uint256 i = 0; i < shares.length; i++ )

226: for( uint256 i = 0; i < rewards.length; i++ )

260: for( uint256 i = 0; i < rewards.length; i++ )

277: for( uint256 i = 0; i < shares.length; i++ )
 
296: for( uint256 i = 0; i < cooldowns.length; i++ )
```
    diff --git a/src/staking/StakingRewards.sol b/src/staking/StakingRewards.sol
    index 246026b..40f2df7 100644
    --- a/src/staking/StakingRewards.sol
    +++ b/src/staking/StakingRewards.sol
    @@ -149,7 +149,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                    mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[msg.sender];

                    claimableRewards = 0;
    -               for( uint256 i = 0; i < poolIDs.length; i++ )
    +               for( uint256 i = poolIDs.length - 1 ; i >= 0 ; i-- )
                            {
                            bytes32 poolID = poolIDs[i];

    @@ -182,7 +182,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
            function addSALTRewards( AddedReward[] calldata addedRewards ) external nonReentrant
                    {
                    uint256 sum = 0;
    -               for( uint256 i = 0; i < addedRewards.length; i++ )
    +               for( uint256 i = addedRewards.length - 1 ; i >= 0 ; i-- )
                            {
                            AddedReward memory addedReward = addedRewards[i];

    @@ -213,7 +213,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                    {
                    shares = new uint256[]( poolIDs.length );

    -               for( uint256 i = 0; i < shares.length; i++ )
    +               for( uint256 i = shares.length - 1; i >= 0 ; i-- )
                            shares[i] = totalShares[ poolIDs[i] ];
                    }

    @@ -223,7 +223,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                    {
                    rewards = new uint256[]( poolIDs.length );

    -               for( uint256 i = 0; i < rewards.length; i++ )
    +               for( uint256 i = rewards.length - 1 ; i >= 0 ; i-- )
                            rewards[i] = totalRewards[ poolIDs[i] ];
                    }

    @@ -257,7 +257,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                    {
                    rewards = new uint256[]( poolIDs.length );

    -               for( uint256 i = 0; i < rewards.length; i++ )
    +               for( uint256 i = rewards.length - 1 ; i >= 0 ; i-- )
                            rewards[i] = userRewardForPool( wallet, poolIDs[i] );
                    }

    @@ -274,7 +274,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                    {
                    shares = new uint256[]( poolIDs.length );

    -               for( uint256 i = 0; i < shares.length; i++ )
    +               for( uint256 i = shares.length - 1; i >= 0 ; i-- )
                            shares[i] = _userShareInfo[wallet][ poolIDs[i] ].userShare;
                    }

    @@ -293,7 +293,7 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard

                    mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[wallet];

    -               for( uint256 i = 0; i < cooldowns.length; i++ )
    +               for( uint256 i = cooldowns.length - 1; i >= 0 ; i-- )
                            {
                            uint256 cooldownExpiration = userInfo[ poolIDs[i] ].cooldownExpiration;

    @@ -303,4 +303,4 @@ abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
                                    cooldowns[i] = cooldownExpiration - block.timestamp;
                            }
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol

```
File: Staking.sol	

163: for(uint256 i = start; i <= end; i++)
```
    diff --git a/src/staking/Staking.sol b/src/staking/Staking.sol
    index 22e5970..c781e62 100644
    --- a/src/staking/Staking.sol
    +++ b/src/staking/Staking.sol
    @@ -160,7 +160,7 @@ contract Staking is IStaking, StakingRewards
             Unstake[] memory unstakes = new Unstake[](end - start + 1);

             uint256 index;
    -        for(uint256 i = start; i <= end; i++)
    +        for(uint256 i = end ; i >= start ; i--)
                 unstakes[index++] = _unstakesByID[ userUnstakes[i]];

             return unstakes;
    @@ -210,4 +210,4 @@ contract Staking is IStaking, StakingRewards
                    uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
            return numerator / ( 100 * unstakeRange );
                    }
    -       }
    \ No newline at end of file
    +       }

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol

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

#### Gas Report

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