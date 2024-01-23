# c4udit Report

## Files analyzed
- 2024-01-salty/src/AccessManager.sol
- 2024-01-salty/src/ExchangeConfig.sol
- 2024-01-salty/src/ManagedWallet.sol
- 2024-01-salty/src/Salt.sol
- 2024-01-salty/src/SigningTools.sol
- 2024-01-salty/src/Upkeep.sol
- 2024-01-salty/src/arbitrage/ArbitrageSearch.sol
- 2024-01-salty/src/dao/DAO.sol
- 2024-01-salty/src/dao/DAOConfig.sol
- 2024-01-salty/src/dao/Parameters.sol
- 2024-01-salty/src/dao/Proposals.sol
- 2024-01-salty/src/dao/interfaces/ICalledContract.sol
- 2024-01-salty/src/dao/interfaces/IDAO.sol
- 2024-01-salty/src/dao/interfaces/IDAOConfig.sol
- 2024-01-salty/src/dao/interfaces/IProposals.sol
- 2024-01-salty/src/interfaces/IAccessManager.sol
- 2024-01-salty/src/interfaces/IExchangeConfig.sol
- 2024-01-salty/src/interfaces/IManagedWallet.sol
- 2024-01-salty/src/interfaces/ISalt.sol
- 2024-01-salty/src/interfaces/IUpkeep.sol
- 2024-01-salty/src/launch/Airdrop.sol
- 2024-01-salty/src/launch/BootstrapBallot.sol
- 2024-01-salty/src/launch/InitialDistribution.sol
- 2024-01-salty/src/launch/interfaces/IAirdrop.sol
- 2024-01-salty/src/launch/interfaces/IBootstrapBallot.sol
- 2024-01-salty/src/launch/interfaces/IInitialDistribution.sol
- 2024-01-salty/src/pools/PoolMath.sol
- 2024-01-salty/src/pools/PoolStats.sol
- 2024-01-salty/src/pools/PoolUtils.sol
- 2024-01-salty/src/pools/Pools.sol
- 2024-01-salty/src/pools/PoolsConfig.sol
- 2024-01-salty/src/pools/interfaces/IPoolStats.sol
- 2024-01-salty/src/pools/interfaces/IPools.sol
- 2024-01-salty/src/pools/interfaces/IPoolsConfig.sol
- 2024-01-salty/src/price_feed/CoreChainlinkFeed.sol
- 2024-01-salty/src/price_feed/CoreSaltyFeed.sol
- 2024-01-salty/src/price_feed/CoreUniswapFeed.sol
- 2024-01-salty/src/price_feed/PriceAggregator.sol
- 2024-01-salty/src/price_feed/interfaces/IPriceAggregator.sol
- 2024-01-salty/src/price_feed/interfaces/IPriceFeed.sol
- 2024-01-salty/src/rewards/Emissions.sol
- 2024-01-salty/src/rewards/RewardsConfig.sol
- 2024-01-salty/src/rewards/RewardsEmitter.sol
- 2024-01-salty/src/rewards/SaltRewards.sol
- 2024-01-salty/src/rewards/interfaces/IEmissions.sol
- 2024-01-salty/src/rewards/interfaces/IRewardsConfig.sol
- 2024-01-salty/src/rewards/interfaces/IRewardsEmitter.sol
- 2024-01-salty/src/rewards/interfaces/ISaltRewards.sol
- 2024-01-salty/src/stable/CollateralAndLiquidity.sol
- 2024-01-salty/src/stable/Liquidizer.sol
- 2024-01-salty/src/stable/StableConfig.sol
- 2024-01-salty/src/stable/USDS.sol
- 2024-01-salty/src/stable/interfaces/ICollateralAndLiquidity.sol
- 2024-01-salty/src/stable/interfaces/ILiquidizer.sol
- 2024-01-salty/src/stable/interfaces/IStableConfig.sol
- 2024-01-salty/src/stable/interfaces/IUSDS.sol
- 2024-01-salty/src/staking/Liquidity.sol
- 2024-01-salty/src/staking/Staking.sol
- 2024-01-salty/src/staking/StakingConfig.sol
- 2024-01-salty/src/staking/StakingRewards.sol
- 2024-01-salty/src/staking/interfaces/ILiquidity.sol
- 2024-01-salty/src/staking/interfaces/IStaking.sol
- 2024-01-salty/src/staking/interfaces/IStakingConfig.sol
- 2024-01-salty/src/staking/interfaces/IStakingRewards.sol

## Issues found

### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
2024-01-salty/src/arbitrage/ArbitrageSearch.sol::115 => for( uint256 i = 0; i < 8; i++ )
2024-01-salty/src/dao/Proposals.sol::421 => uint256 bestID = 0;
2024-01-salty/src/dao/Proposals.sol::422 => uint256 mostYes = 0;
2024-01-salty/src/dao/Proposals.sol::423 => for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
2024-01-salty/src/pools/PoolMath.sol::156 => uint256 shift = 0;
2024-01-salty/src/pools/PoolStats.sol::53 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::65 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::81 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::106 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/price_feed/CoreUniswapFeed.sol::82 => uint256 twap = 0;
2024-01-salty/src/rewards/RewardsEmitter.sol::59 => uint256 sum = 0;
2024-01-salty/src/rewards/RewardsEmitter.sol::60 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/RewardsEmitter.sol::115 => uint256 sum = 0;
2024-01-salty/src/rewards/RewardsEmitter.sol::116 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/rewards/RewardsEmitter.sol::146 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::64 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::87 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::128 => uint256 totalProfits = 0;
2024-01-salty/src/rewards/SaltRewards.sol::129 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/stable/CollateralAndLiquidity.sol::314 => uint256 count = 0;
2024-01-salty/src/stable/CollateralAndLiquidity.sol::338 => for ( uint256 i = 0; i < count; i++ )
2024-01-salty/src/staking/StakingRewards.sol::128 => uint256 claimableRewards = 0;
2024-01-salty/src/staking/StakingRewards.sol::152 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::184 => uint256 sum = 0;
2024-01-salty/src/staking/StakingRewards.sol::185 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::216 => for( uint256 i = 0; i < shares.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::226 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::260 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::277 => for( uint256 i = 0; i < shares.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::296 => for( uint256 i = 0; i < cooldowns.length; i++ )
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
2024-01-salty/src/SigningTools.sol::13 => require( signature.length == 65, "Invalid signature length" );
2024-01-salty/src/dao/Proposals.sol::167 => require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );
2024-01-salty/src/dao/Proposals.sol::224 => require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
2024-01-salty/src/dao/Proposals.sol::233 => require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
2024-01-salty/src/dao/Proposals.sol::423 => for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
2024-01-salty/src/launch/Airdrop.sol::99 => return _authorizedUsers.length();
2024-01-salty/src/pools/PoolStats.sol::53 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::65 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::81 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::106 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/pools/PoolStats.sol::138 => _calculatedProfits = new uint256[](poolIDs.length);
2024-01-salty/src/pools/PoolsConfig.sol::47 => require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
2024-01-salty/src/pools/PoolsConfig.sol::115 => return _whitelist.length();
2024-01-salty/src/rewards/RewardsEmitter.sol::60 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/RewardsEmitter.sol::109 => AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
2024-01-salty/src/rewards/RewardsEmitter.sol::116 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/rewards/RewardsEmitter.sol::144 => uint256[] memory rewards = new uint256[]( pools.length );
2024-01-salty/src/rewards/RewardsEmitter.sol::146 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::60 => require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
2024-01-salty/src/rewards/SaltRewards.sol::63 => AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
2024-01-salty/src/rewards/SaltRewards.sol::64 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::84 => uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length; // poolIDs.length is guaranteed to not be zero
2024-01-salty/src/rewards/SaltRewards.sol::86 => AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
2024-01-salty/src/rewards/SaltRewards.sol::87 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/rewards/SaltRewards.sol::120 => require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
2024-01-salty/src/rewards/SaltRewards.sol::129 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/stable/CollateralAndLiquidity.sol::292 => return _walletsWithBorrowedUSDS.length();
2024-01-salty/src/staking/Staking.sol::157 => require(userUnstakes.length > end, "Invalid range: end is out of bounds");
2024-01-salty/src/staking/Staking.sol::158 => require(start < userUnstakes.length, "Invalid range: start is out of bounds");
2024-01-salty/src/staking/Staking.sol::175 => if ( unstakeIDs.length == 0 )
2024-01-salty/src/staking/Staking.sol::179 => return unstakesForUser( user, 0, unstakeIDs.length - 1 );
2024-01-salty/src/staking/StakingRewards.sol::152 => for( uint256 i = 0; i < poolIDs.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::185 => for( uint256 i = 0; i < addedRewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::214 => shares = new uint256[]( poolIDs.length );
2024-01-salty/src/staking/StakingRewards.sol::216 => for( uint256 i = 0; i < shares.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::224 => rewards = new uint256[]( poolIDs.length );
2024-01-salty/src/staking/StakingRewards.sol::226 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::258 => rewards = new uint256[]( poolIDs.length );
2024-01-salty/src/staking/StakingRewards.sol::260 => for( uint256 i = 0; i < rewards.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::275 => shares = new uint256[]( poolIDs.length );
2024-01-salty/src/staking/StakingRewards.sol::277 => for( uint256 i = 0; i < shares.length; i++ )
2024-01-salty/src/staking/StakingRewards.sol::292 => cooldowns = new uint256[]( poolIDs.length );
2024-01-salty/src/staking/StakingRewards.sol::296 => for( uint256 i = 0; i < cooldowns.length; i++ )
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
2024-01-salty/src/dao/Proposals.sol::92 => require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
2024-01-salty/src/dao/Proposals.sol::277 => require( userVotingPower > 0, "Staked SALT required to vote" );
2024-01-salty/src/dao/Proposals.sol::283 => if ( lastVote.votingPower > 0 )
2024-01-salty/src/launch/Airdrop.sol::60 => require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
2024-01-salty/src/launch/BootstrapBallot.sol::36 => require( ballotDuration > 0, "ballotDuration cannot be zero" );
2024-01-salty/src/pools/PoolStats.sol::113 => if ( arbitrageProfit > 0 )
2024-01-salty/src/pools/Pools.sol::173 => require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
2024-01-salty/src/pools/Pools.sol::340 => if (arbitrageAmountIn > 0)
2024-01-salty/src/price_feed/PriceAggregator.sol::112 => if (price1 > 0)
2024-01-salty/src/price_feed/PriceAggregator.sol::115 => if (price2 > 0)
2024-01-salty/src/price_feed/PriceAggregator.sol::118 => if (price3 > 0)
2024-01-salty/src/rewards/RewardsEmitter.sol::75 => if ( sum > 0 )
2024-01-salty/src/stable/CollateralAndLiquidity.sol::83 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::98 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::117 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::119 => require( amountRepaid > 0, "Cannot repay zero amount" );
2024-01-salty/src/stable/Liquidizer.sol::145 => if ( saltBalance > 0 )
2024-01-salty/src/stable/USDS.sol::43 => require( amount > 0, "Cannot mint zero USDS" );
2024-01-salty/src/staking/Liquidity.sol::56 => if ( swapAmountA > 0)
2024-01-salty/src/staking/Liquidity.sol::66 => else if ( swapAmountB > 0)
2024-01-salty/src/staking/Staking.sol::115 => if ( expeditedUnstakeFee > 0 )
2024-01-salty/src/staking/StakingRewards.sol::164 => if ( claimableRewards > 0 )
2024-01-salty/src/staking/StakingRewards.sol::201 => if ( sum > 0 )
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G006](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g006---use-immutable-for-openzeppelin-accesscontrols-roles-declarations)

#### Findings:
```
2024-01-salty/src/AccessManager.sol::53 => bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
2024-01-salty/src/dao/DAO.sol::133 => bytes32 nameHash = keccak256(bytes( ballot.ballotName ) );
2024-01-salty/src/dao/DAO.sol::135 => if ( nameHash == keccak256(bytes( "setContract:priceFeed1_confirm" )) )
2024-01-salty/src/dao/DAO.sol::137 => else if ( nameHash == keccak256(bytes( "setContract:priceFeed2_confirm" )) )
2024-01-salty/src/dao/DAO.sol::139 => else if ( nameHash == keccak256(bytes( "setContract:priceFeed3_confirm" )) )
2024-01-salty/src/dao/DAO.sol::141 => else if ( nameHash == keccak256(bytes( "setContract:accessManager_confirm" )) )
2024-01-salty/src/dao/Proposals.sol::251 => require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
2024-01-salty/src/launch/BootstrapBallot.sol::53 => bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
2024-01-salty/src/pools/PoolUtils.sol::25 => return keccak256(abi.encodePacked(address(tokenB), address(tokenA)));
2024-01-salty/src/pools/PoolUtils.sol::27 => return keccak256(abi.encodePacked(address(tokenA), address(tokenB)));
2024-01-salty/src/pools/PoolUtils.sol::36 => return (keccak256(abi.encodePacked(address(tokenB), address(tokenA))), true);
2024-01-salty/src/pools/PoolUtils.sol::38 => return (keccak256(abi.encodePacked(address(tokenA), address(tokenB))), false);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
2024-01-salty/src/AccessManager.sol::43 => require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
2024-01-salty/src/AccessManager.sol::63 => require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );
2024-01-salty/src/ExchangeConfig.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/ExchangeConfig.sol::5 => import "./launch/interfaces/IInitialDistribution.sol";
2024-01-salty/src/ExchangeConfig.sol::6 => import "./rewards/interfaces/IRewardsEmitter.sol";
2024-01-salty/src/ExchangeConfig.sol::51 => require( address(dao) == address(0), "setContracts can only be called once" );
2024-01-salty/src/ExchangeConfig.sol::64 => require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" );
2024-01-salty/src/ManagedWallet.sol::44 => require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
2024-01-salty/src/ManagedWallet.sol::45 => require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
2024-01-salty/src/ManagedWallet.sol::46 => require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
2024-01-salty/src/ManagedWallet.sol::49 => require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );
2024-01-salty/src/Salt.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/Upkeep.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/Upkeep.sol::5 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/Upkeep.sol::6 => import "openzeppelin-contracts/contracts/finance/VestingWallet.sol";
2024-01-salty/src/Upkeep.sol::7 => import "./price_feed/interfaces/IPriceAggregator.sol";
2024-01-salty/src/Upkeep.sol::8 => import "./stable/interfaces/IStableConfig.sol";
2024-01-salty/src/Upkeep.sol::9 => import "./rewards/interfaces/IEmissions.sol";
2024-01-salty/src/Upkeep.sol::10 => import "./pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/Upkeep.sol::97 => require(msg.sender == address(this), "Only callable from within the same contract");
2024-01-salty/src/arbitrage/ArbitrageSearch.sol::4 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/dao/DAO.sol::4 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/dao/DAO.sol::5 => import "../price_feed/interfaces/IPriceAggregator.sol";
2024-01-salty/src/dao/DAO.sol::6 => import "../rewards/interfaces/IRewardsEmitter.sol";
2024-01-salty/src/dao/DAO.sol::7 => import "../rewards/interfaces/IRewardsConfig.sol";
2024-01-salty/src/dao/DAO.sol::8 => import "../stable/interfaces/IStableConfig.sol";
2024-01-salty/src/dao/DAO.sol::9 => import "../stable/interfaces/ILiquidizer.sol";
2024-01-salty/src/dao/DAO.sol::10 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/dao/DAO.sol::11 => import "../staking/interfaces/IStaking.sol";
2024-01-salty/src/dao/DAO.sol::141 => else if ( nameHash == keccak256(bytes( "setContract:accessManager_confirm" )) )
2024-01-salty/src/dao/DAO.sol::204 => proposals.createConfirmationProposal( string.concat(ballot.ballotName, "_confirm"), BallotType.CONFIRM_SET_CONTRACT, ballot.address1, "", ballot.description );
2024-01-salty/src/dao/DAO.sol::247 => require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
2024-01-salty/src/dao/DAO.sol::251 => require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );
2024-01-salty/src/dao/DAO.sol::281 => require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );
2024-01-salty/src/dao/DAO.sol::297 => require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );
2024-01-salty/src/dao/DAO.sol::318 => require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );
2024-01-salty/src/dao/DAO.sol::329 => require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );
2024-01-salty/src/dao/DAO.sol::362 => require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
2024-01-salty/src/dao/DAOConfig.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/dao/Parameters.sol::4 => import "../price_feed/interfaces/IPriceAggregator.sol";
2024-01-salty/src/dao/Parameters.sol::5 => import "../rewards/interfaces/IRewardsConfig.sol";
2024-01-salty/src/dao/Parameters.sol::6 => import "../staking/interfaces/IStakingConfig.sol";
2024-01-salty/src/dao/Parameters.sol::7 => import "../stable/interfaces/IStableConfig.sol";
2024-01-salty/src/dao/Parameters.sol::8 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/dao/Proposals.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/dao/Proposals.sol::5 => import "openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol";
2024-01-salty/src/dao/Proposals.sol::6 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/dao/Proposals.sol::7 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/dao/Proposals.sol::8 => import "openzeppelin-contracts/contracts/utils/Strings.sol";
2024-01-salty/src/dao/Proposals.sol::9 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/dao/Proposals.sol::10 => import "../staking/interfaces/IStaking.sol";
2024-01-salty/src/dao/Proposals.sol::11 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/dao/Proposals.sol::83 => require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );
2024-01-salty/src/dao/Proposals.sol::95 => require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );
2024-01-salty/src/dao/Proposals.sol::98 => require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
2024-01-salty/src/dao/Proposals.sol::102 => require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
2024-01-salty/src/dao/Proposals.sol::103 => require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );
2024-01-salty/src/dao/Proposals.sol::124 => require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );
2024-01-salty/src/dao/Proposals.sol::132 => require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
2024-01-salty/src/dao/Proposals.sol::165 => require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision
2024-01-salty/src/dao/Proposals.sol::167 => require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );
2024-01-salty/src/dao/Proposals.sol::168 => require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );
2024-01-salty/src/dao/Proposals.sol::169 => require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );
2024-01-salty/src/dao/Proposals.sol::182 => require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
2024-01-salty/src/dao/Proposals.sol::203 => require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );
2024-01-salty/src/dao/Proposals.sol::215 => require( contractAddress != address(0), "Contract address cannot be address(0)" );
2024-01-salty/src/dao/Proposals.sol::224 => require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
2024-01-salty/src/dao/Proposals.sol::233 => require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
2024-01-salty/src/dao/Proposals.sol::242 => require( newAddress != address(0), "Proposed address cannot be address(0)" );
2024-01-salty/src/dao/Proposals.sol::251 => require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
2024-01-salty/src/dao/Proposals.sol::264 => require( ballot.ballotIsLive, "The specified ballot is not open for voting" );
2024-01-salty/src/dao/Proposals.sol::268 => require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
2024-01-salty/src/dao/Proposals.sol::270 => require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
2024-01-salty/src/dao/Proposals.sol::321 => require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );
2024-01-salty/src/dao/interfaces/IDAO.sol::4 => import "../../rewards/interfaces/ISaltRewards.sol";
2024-01-salty/src/dao/interfaces/IDAO.sol::5 => import "../../stable/interfaces/IUSDS.sol";
2024-01-salty/src/dao/interfaces/IDAO.sol::6 => import "../../pools/interfaces/IPools.sol";
2024-01-salty/src/dao/interfaces/IProposals.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::4 => import "openzeppelin-contracts/contracts/finance/VestingWallet.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::5 => import "../stable/interfaces/ICollateralAndLiquidity.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::6 => import "../launch/interfaces/IInitialDistribution.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::7 => import "../rewards/interfaces/IRewardsEmitter.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::8 => import "../rewards/interfaces/ISaltRewards.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::9 => import "../rewards/interfaces/IEmissions.sol";
2024-01-salty/src/interfaces/IExchangeConfig.sol::11 => import "../launch/interfaces/IAirdrop.sol";
2024-01-salty/src/interfaces/ISalt.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/launch/Airdrop.sol::4 => import "openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol";
2024-01-salty/src/launch/Airdrop.sol::5 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/launch/Airdrop.sol::6 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/launch/Airdrop.sol::7 => import "../staking/interfaces/IStaking.sol";
2024-01-salty/src/launch/Airdrop.sol::48 => require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
2024-01-salty/src/launch/Airdrop.sol::49 => require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );
2024-01-salty/src/launch/Airdrop.sol::58 => require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
2024-01-salty/src/launch/Airdrop.sol::60 => require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
2024-01-salty/src/launch/Airdrop.sol::77 => require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
2024-01-salty/src/launch/Airdrop.sol::78 => require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );
2024-01-salty/src/launch/BootstrapBallot.sol::4 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/launch/BootstrapBallot.sol::5 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/launch/BootstrapBallot.sol::6 => import "./interfaces/IBootstrapBallot.sol";
2024-01-salty/src/launch/BootstrapBallot.sol::54 => require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );
2024-01-salty/src/launch/BootstrapBallot.sol::71 => require( ! ballotFinalized, "Ballot has already been finalized" );
2024-01-salty/src/launch/InitialDistribution.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/launch/InitialDistribution.sol::5 => import "openzeppelin-contracts/contracts/finance/VestingWallet.sol";
2024-01-salty/src/launch/InitialDistribution.sol::6 => import "../rewards/interfaces/ISaltRewards.sol";
2024-01-salty/src/launch/InitialDistribution.sol::7 => import "../rewards/interfaces/IEmissions.sol";
2024-01-salty/src/launch/InitialDistribution.sol::8 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/launch/InitialDistribution.sol::9 => import "./interfaces/IInitialDistribution.sol";
2024-01-salty/src/launch/InitialDistribution.sol::10 => import "./interfaces/IBootstrapBallot.sol";
2024-01-salty/src/launch/InitialDistribution.sol::52 => require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
2024-01-salty/src/launch/InitialDistribution.sol::53 => require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
2024-01-salty/src/pools/PoolMath.sol::3 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/pools/PoolMath.sol::4 => import "openzeppelin-contracts/contracts/utils/math/Math.sol";
2024-01-salty/src/pools/PoolStats.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/pools/PoolStats.sol::5 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/pools/PoolStats.sol::49 => require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
2024-01-salty/src/pools/PoolUtils.sol::3 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/pools/PoolUtils.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/pools/PoolUtils.sol::5 => import "openzeppelin-contracts/contracts/utils/math/Math.sol";
2024-01-salty/src/pools/Pools.sol::4 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/pools/Pools.sol::5 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/pools/Pools.sol::6 => import "openzeppelin-contracts/contracts/utils/math/Math.sol";
2024-01-salty/src/pools/Pools.sol::7 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/pools/Pools.sol::8 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/pools/Pools.sol::72 => require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );
2024-01-salty/src/pools/Pools.sol::142 => require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
2024-01-salty/src/pools/Pools.sol::144 => require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );
2024-01-salty/src/pools/Pools.sol::146 => require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
2024-01-salty/src/pools/Pools.sol::147 => require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );
2024-01-salty/src/pools/Pools.sol::172 => require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
2024-01-salty/src/pools/Pools.sol::173 => require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
2024-01-salty/src/pools/Pools.sol::187 => require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
2024-01-salty/src/pools/Pools.sol::193 => require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );
2024-01-salty/src/pools/Pools.sol::221 => require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
2024-01-salty/src/pools/Pools.sol::243 => require((reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves before swap");
2024-01-salty/src/pools/Pools.sol::281 => // Essentially the caller virtually "borrows" arbitrageAmountIn of the starting token and virtually "repays" it from their received amountOut at the end of the swap chain.
2024-01-salty/src/pools/Pools.sol::353 => require( swapAmountOut >= minAmountOut, "Insufficient resulting token amount" );
2024-01-salty/src/pools/Pools.sol::371 => require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" );
2024-01-salty/src/pools/PoolsConfig.sol::4 => import "openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol";
2024-01-salty/src/pools/PoolsConfig.sol::5 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/pools/PoolsConfig.sol::47 => require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
2024-01-salty/src/pools/PoolsConfig.sol::48 => require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
2024-01-salty/src/pools/interfaces/IPools.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/pools/interfaces/IPools.sol::5 => import "../../stable/interfaces/ICollateralAndLiquidity.sol";
2024-01-salty/src/pools/interfaces/IPoolsConfig.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/price_feed/CoreChainlinkFeed.sol::4 => import "chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
2024-01-salty/src/price_feed/CoreSaltyFeed.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/price_feed/CoreSaltyFeed.sol::5 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/price_feed/CoreUniswapFeed.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/price_feed/CoreUniswapFeed.sol::5 => import "v3-core/interfaces/IUniswapV3Pool.sol";
2024-01-salty/src/price_feed/CoreUniswapFeed.sol::6 => import "v3-core/libraries/FixedPoint96.sol";
2024-01-salty/src/price_feed/PriceAggregator.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/price_feed/PriceAggregator.sol::6 => import "./interfaces/IPriceAggregator.sol";
2024-01-salty/src/price_feed/PriceAggregator.sol::39 => require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );
2024-01-salty/src/rewards/Emissions.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/rewards/Emissions.sol::5 => import "../rewards/interfaces/IRewardsConfig.sol";
2024-01-salty/src/rewards/Emissions.sol::6 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/rewards/Emissions.sol::42 => require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
2024-01-salty/src/rewards/RewardsConfig.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::5 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::6 => import "../rewards/interfaces/IRewardsConfig.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::7 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::8 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/rewards/RewardsEmitter.sol::84 => require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
2024-01-salty/src/rewards/SaltRewards.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/rewards/SaltRewards.sol::5 => import "../rewards/interfaces/IRewardsEmitter.sol";
2024-01-salty/src/rewards/SaltRewards.sol::6 => import "../rewards/interfaces/IRewardsConfig.sol";
2024-01-salty/src/rewards/SaltRewards.sol::7 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/rewards/SaltRewards.sol::108 => require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );
2024-01-salty/src/rewards/SaltRewards.sol::119 => require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
2024-01-salty/src/rewards/interfaces/IRewardsEmitter.sol::4 => import "../../staking/interfaces/IStakingRewards.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::5 => import "openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::6 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::7 => import "openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Metadata.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::8 => import "../price_feed/interfaces/IPriceAggregator.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::9 => import "./interfaces/ICollateralAndLiquidity.sol";
2024-01-salty/src/stable/CollateralAndLiquidity.sol::83 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::97 => require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::98 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::117 => require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
2024-01-salty/src/stable/CollateralAndLiquidity.sol::118 => require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
2024-01-salty/src/stable/Liquidizer.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/stable/Liquidizer.sol::5 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/stable/Liquidizer.sol::6 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/stable/Liquidizer.sol::7 => import "../stable/interfaces/ICollateralAndLiquidity.sol";
2024-01-salty/src/stable/Liquidizer.sol::8 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/stable/Liquidizer.sol::9 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/stable/Liquidizer.sol::83 => require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
2024-01-salty/src/stable/Liquidizer.sol::134 => require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
2024-01-salty/src/stable/StableConfig.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/stable/USDS.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
2024-01-salty/src/stable/USDS.sol::5 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/stable/USDS.sol::6 => import "../stable/interfaces/ICollateralAndLiquidity.sol";
2024-01-salty/src/stable/USDS.sol::42 => require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
2024-01-salty/src/stable/interfaces/ICollateralAndLiquidity.sol::4 => import "../../staking/interfaces/ILiquidity.sol";
2024-01-salty/src/stable/interfaces/ILiquidizer.sol::4 => import "../../interfaces/IExchangeConfig.sol";
2024-01-salty/src/stable/interfaces/ILiquidizer.sol::5 => import "../../pools/interfaces/IPools.sol";
2024-01-salty/src/stable/interfaces/IUSDS.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
2024-01-salty/src/staking/Liquidity.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/staking/Liquidity.sol::5 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/staking/Liquidity.sol::6 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/staking/Liquidity.sol::85 => require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
2024-01-salty/src/staking/Liquidity.sol::124 => require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );
2024-01-salty/src/staking/Liquidity.sol::148 => require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be deposited via Liquidity.depositLiquidityAndIncreaseShare" );
2024-01-salty/src/staking/Liquidity.sol::159 => require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be withdrawn via Liquidity.withdrawLiquidityAndClaim" );
2024-01-salty/src/staking/Staking.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/staking/Staking.sol::43 => require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
2024-01-salty/src/staking/Staking.sol::62 => require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" );
2024-01-salty/src/staking/Staking.sol::88 => require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" );
2024-01-salty/src/staking/Staking.sol::89 => require( block.timestamp < u.completionTime, "Unstakes that have already completed cannot be cancelled" );
2024-01-salty/src/staking/Staking.sol::90 => require( msg.sender == u.wallet, "Sender is not the original staker" );
2024-01-salty/src/staking/Staking.sol::104 => require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" );
2024-01-salty/src/staking/Staking.sol::106 => require( msg.sender == u.wallet, "Sender is not the original staker" );
2024-01-salty/src/staking/Staking.sol::132 => require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );
2024-01-salty/src/staking/Staking.sol::153 => require(end >= start, "Invalid range: end cannot be less than start");
2024-01-salty/src/staking/Staking.sol::157 => require(userUnstakes.length > end, "Invalid range: end is out of bounds");
2024-01-salty/src/staking/Staking.sol::158 => require(start < userUnstakes.length, "Invalid range: start is out of bounds");
2024-01-salty/src/staking/StakingConfig.sol::4 => import "openzeppelin-contracts/contracts/access/Ownable.sol";
2024-01-salty/src/staking/StakingRewards.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
2024-01-salty/src/staking/StakingRewards.sol::5 => import "openzeppelin-contracts/contracts/security/ReentrancyGuard.sol";
2024-01-salty/src/staking/StakingRewards.sol::6 => import "openzeppelin-contracts/contracts/utils/math/Math.sol";
2024-01-salty/src/staking/StakingRewards.sol::7 => import "../pools/interfaces/IPoolsConfig.sol";
2024-01-salty/src/staking/StakingRewards.sol::8 => import "../interfaces/IExchangeConfig.sol";
2024-01-salty/src/staking/StakingRewards.sol::67 => require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
2024-01-salty/src/staking/StakingRewards.sol::102 => require( decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share" );
2024-01-salty/src/staking/StakingRewards.sol::107 => require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
2024-01-salty/src/staking/interfaces/ILiquidity.sol::4 => import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
2024-01-salty/src/ManagedWallet.sol::9 => // 2. Confirmation wallet confirms or rejects.
2024-01-salty/src/Upkeep.sol::22 => // 2. Withdraws existing WETH arbitrage profits from the Pools contract and rewards the caller of performUpkeep() with default 5% of the withdrawn amount.
2024-01-salty/src/Upkeep.sol::24 => // 4. Converts a default 20% of the remaining WETH to SALT/USDS Protocol Owned Liquidity.
2024-01-salty/src/Upkeep.sol::29 => // 8. Distributes SALT rewards from the stakingRewardsEmitter and liquidityRewardsEmitter.
2024-01-salty/src/Upkeep.sol::111 => // 2. Withdraw existing WETH arbitrage profits from the Pools contract and reward the caller of performUpkeep() with default 5% of the withdrawn amount.
2024-01-salty/src/Upkeep.sol::157 => // 4. Convert a default 20% of the remaining WETH to SALT/USDS Protocol Owned Liquidity.
2024-01-salty/src/Upkeep.sol::204 => // 8. Distribute SALT rewards from the stakingRewardsEmitter and liquidityRewardsEmitter.
2024-01-salty/src/dao/DAO.sol::247 => require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
2024-01-salty/src/dao/DAO.sol::265 => exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
2024-01-salty/src/launch/InitialDistribution.sol::58 => // 25 million		DAO Reserve Vesting Wallet
2024-01-salty/src/pools/PoolMath.sol::111 => x = [-B + sqrt(B^2 - 4AC)] / 2A
2024-01-salty/src/pools/PoolMath.sol::123 => if (x >= 2**8) { x >>= 8; msb += 8; }
2024-01-salty/src/pools/PoolMath.sol::124 => if (x >= 2**4) { x >>= 4; msb += 4; }
2024-01-salty/src/pools/PoolMath.sol::125 => if (x >= 2**2) { x >>= 2; msb += 2; }
2024-01-salty/src/pools/PoolMath.sol::182 => // Components of the quadratic formula mentioned in the initial comment block: x = [-B + sqrt(B^2 - 4AC)] / 2A
2024-01-salty/src/pools/PoolMath.sol::202 => // Only use the positive sqrt of the discriminant from: x = (-B +/- sqrtDiscriminant) / 2A
2024-01-salty/src/price_feed/CoreSaltyFeed.sol::40 => return ( reservesUSDS * 10**8 ) / reservesWBTC;
2024-01-salty/src/price_feed/PriceAggregator.sol::139 => uint256 averagePrice = ( priceA + priceB ) / 2;
2024-01-salty/src/staking/StakingRewards.sol::19 => // 2. Liquidity.sol: shares represent the amount of liquidity deposited and staked to specific pools
2024-01-salty/src/staking/StakingRewards.sol::177 => // 2. Staked SALT has a default unstake period of 52 weeks.
2024-01-salty/src/staking/StakingRewards.sol::179 => // 4. Rewards are deposited fairly often, with outstanding rewards being transferred with a frequency proportional to the activity of the exchange.
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
2024-01-salty/src/Upkeep.sol::91 => weth.approve( address(pools), type(uint256).max );
2024-01-salty/src/dao/DAO.sol::90 => salt.approve(address(collateralAndLiquidity), type(uint256).max);
2024-01-salty/src/dao/DAO.sol::91 => usds.approve(address(collateralAndLiquidity), type(uint256).max);
2024-01-salty/src/dao/DAO.sol::92 => dai.approve(address(collateralAndLiquidity), type(uint256).max);
2024-01-salty/src/dao/DAO.sol::265 => exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
2024-01-salty/src/launch/Airdrop.sol::67 => salt.approve( address(staking), saltBalance );
2024-01-salty/src/rewards/RewardsEmitter.sol::50 => salt.approve(address(stakingRewards), type(uint256).max);
2024-01-salty/src/rewards/SaltRewards.sol::41 => salt.approve( address(stakingRewardsEmitter), type(uint256).max );
2024-01-salty/src/rewards/SaltRewards.sol::42 => salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
2024-01-salty/src/stable/Liquidizer.sol::71 => wbtc.approve( address(pools), type(uint256).max );
2024-01-salty/src/stable/Liquidizer.sol::72 => weth.approve( address(pools), type(uint256).max );
2024-01-salty/src/stable/Liquidizer.sol::73 => dai.approve( address(pools), type(uint256).max );
2024-01-salty/src/staking/Liquidity.sol::58 => tokenA.approve( address(pools), swapAmountA );
2024-01-salty/src/staking/Liquidity.sol::68 => tokenB.approve( address(pools), swapAmountB );
2024-01-salty/src/staking/Liquidity.sol::96 => tokenA.approve( address(pools), maxAmountA );
2024-01-salty/src/staking/Liquidity.sol::97 => tokenB.approve( address(pools), maxAmountB );
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)
