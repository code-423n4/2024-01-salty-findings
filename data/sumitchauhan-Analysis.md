High risk issues:
1. Reentrancy
CollateralAndLiquidity.sol::140-188 Reentrancy is present in liquidateUser(address) function due to external calls sending eth and state variables written after the calls.
Liquidizer.sol::101-126 Reentrancy is present in _possiblyBurnUSDS() function due to external calls sending eth and state variables written after the calls.
Staking.sol::130-138 Reentrancy is present in transferStakedSaltFromAirdropToUser(address,uint256) function due to external calls sending eth and state variables written after the calls.
      
Medium risk issues:
1.Dangerous strict equalities
SaltRewards.sol::124 The function performUpkeep(bytes32[],uint256[]) uses a dangerous strict equality i.e., saltRewardsToDistribute == 0 
CollateralAndLiquidity.sol::271 maxBorrowableUSDS(address) uses a dangerous strict equality i.e., if ( userShareForPool( wallet, collateralPoolID ) == 0 ) 
CollateralAndLiquidity.sol::271 maxWithdrawableCollateral(address) uses a dangerous strict equality i.e., if ( userCollateralAmount == 0 )
CollateralAndLiquidity.sol::224 userCollateralValueInUSD(address) uses a dangerous strict equality i.e., if ( userCollateralAmount == 0 )
StakingRewards.sol::235 userRewardForPool(address,bytes32) uses a dangerous strict equality i.e., totalShares[poolID] == 0 
StakingRewards.sol::239 userRewardForPool(address,bytes32) uses a dangerous strict equality i.e., if ( user.userShare == 0 )
Upkeep.sol::148 The function step3() uses a dangerous strict equality i.e., wethBalance == 0 
Upkeep.sol::161 The function step4() uses a dangerous strict equality i.e., wethBalance == 0 
Upkeep.sol::174 The function step5() uses a dangerous strict equality i.e., wethBalance == 0 
Proposals.sol::317-339 The function requiredQuorumForBallotType( BallotType ballotType ) uses a dangerous strict equality i.e., else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
InitialDistribution.sol::50-75 The function distributionApproved() uses a dangerous strict equality i.e., require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );

2. Locked ether
ManagedWallet.sol::59-69 ManagedWallet.sol contract has a payable receive() function but does not have a function to withdraw the ether.

Low risk issues:
1. Incorrect versions of solidity
Pragma version0.8.22 necessitates a version too recent to be trusted. Consider deploying with 0.8.18. solc-0.8.22 is not recommended for deployment.

2. Public variable read in external context
CoreUniswapFeed.sol::50-73 The function getUniswapTwapWei(IUniswapV3Pool,uint256) reads result = this._getUniswapTwapWei(pool,twapInterval) with `this` which adds an extra STATICCALL.

Informational issues or gas optimizations:
1. Usage of too many digits
RewardsEmitter.sol::113 The function performUpkeep(uint256) uses literals with too many digits i.e., uint256 denominatorMult = 1 days * 100000
Salt.sol::13 The variable INITIAL_SUPPLY uses literals with too many digits i.e., uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;
PoolUtils.sol::61  The variable maxAmountIN uses literals with too many digits i.e., uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
 

2. Use of block.timestamp
- Following functions in CollateralAndLiquidity.sol uses timestamp for comparisons-
withdrawCollateralAndClaim(uint256,uint256,uint256,uint256), 
borrowUSDS(uint256),
repayUSDS(uint256), 
liquidateUser(address), 
userCollateralValueInUSD(address), 
maxWithdrawableCollateral(address), 
maxBorrowableUSDS(address), 
canUserBeLiquidated(address), 
findLiquidatableUsers(uint256,uint256)

- Following functions in StakingRewards.sol uses timestamp for comparisons-
_decreaseUserShare(address,bytes32,uint256,bool),
userCooldowns(address,bytes32[]),
unstake(uint256,uint256),
cancelUnstake(uint256),
recoverSALT(uint256)

- Following functions in Proposals.sol uses timestamp for comparisons-       
_possiblyCreateProposal(string,BallotType,address,uint256,string,string),
requiredQuorumForBallotType(BallotType),
totalVotesCastForBallot(uint256),
canFinalizeBallot(uint256) 

- latestChainlinkPrice(AggregatorV3Interface chainlinkFeed) function in CoreChainlinkFeed uses timestamp for comparisons.



### Time spent:
40 hours