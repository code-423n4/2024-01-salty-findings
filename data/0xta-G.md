# Gas Optimization

## [G-01] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: src/stable/CollateralAndLiquidity.sol

350		return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L350

```solidity
File: src/stable/StableConfig.sol

52			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

128			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L52

```solidity
File: src/staking/Staking.sol

160        Unstake[] memory unstakes = new Unstake[](end - start + 1);

179		return unstakesForUser( user, 0, unstakeIDs.length - 1 );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L160

```solidity
File: src/stable/CollateralAndLiquidity.sol

313		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L313

## [G-02] Use ASSEMBLY to validate msg.sender

```solidity
File: src/AccessManager.sol

43    	require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43

```solidity
File: src/launch/Airdrop.sol

48    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

58    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48

```solidity
File: src/stable/CollateralAndLiquidity.sol

142		require( wallet != msg.sender, "Cannot liquidate self" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L142

```solidity
File: src/dao/DAO.sol

297		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

318		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

362		require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297

```solidity
File: src/rewards/Emissions.sol

42		require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42

```solidity
File: /home/x0w3r/progress-ON/2024-01-salty/scope/InitialDistribution.sol

52    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol

```solidity
File: src/stable/Liquidizer.sol

83		require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

134		require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83

```solidity
File: src/ManagedWallet.sol

44		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

61    	require( msg.sender == confirmationWallet, "Invalid sender" );

76		require( msg.sender == proposedMainWallet, "Invalid sender" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44

```solidity
File: src/pools/Pools.sol

72    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

142		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

172		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72

```solidity
File: src/dao/Proposals.sol

86		if ( msg.sender != address(exchangeConfig.dao() ) )

124		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L86

```solidity
File: src/rewards/SaltRewards.sol

108		require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );


119		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108

```solidity
File: src/staking/StakingRewards.sol

65		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown

105		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L65

## [G-03] Access MAPPINGS directly rather than using accessor functions

```solidity
File:  src/AccessManager.sol

74    function walletHasAccess(address wallet) external view returns (bool)
75    	{
76        return _walletsWithAccess[geoVersion][wallet];
77    	}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L74

```solidity
File: src/dao/DAO.sol


384	function countryIsExcluded( string calldata country ) external view returns (bool)
385		{
386		return excludedCountries[country];
387		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L384

```solidity
File: src/pools/Pools.sol


423	function depositedUserBalance(address user, IERC20 token) public view returns (uint256)
424		{
425		return _userDeposits[user][token];
426		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L423

```solidity
File: pools/PoolStats.sol


143	function arbitrageIndicies(bytes32 poolID) external view returns (ArbitrageIndicies memory)
144		{
145		return _arbitrageIndicies[poolID];
146		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L143

```solidity
File: src/dao/Proposals.sol

442	function userHasActiveProposal( address user ) external view returns (bool)
443		{
444		return _userHasActiveProposal[user];
445		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L442

```solidity
File: src/staking/Staking.sol

184	function userUnstakeIDs( address user ) external view returns (uint256[] memory)
185		{
186		return _userUnstakeIDs[user];
187		}


190	function unstakeByID(uint256 id) external view returns (Unstake memory)
191		{
192		return _unstakesByID[id];
193		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L184

## [G-04] Use assembly to reuse memory space when making more than one external call

```solidity
File: src/launch/Airdrop.sol

81		staking.stakeSALT( saltAmountForEachUser );
82		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L81

```solidity
File: src/launch/BootstrapBallot.sol

76			exchangeConfig.initialDistribution().distributionApproved();
77			exchangeConfig.dao().pools().startExchangeApproved();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L76

```solidity
File: src/stable/CollateralAndLiquidity.sol

172		wbtc.safeTransfer( msg.sender, rewardedWBTC );
173		weth.safeTransfer( msg.sender, rewardedWETH );

		Liquidizer.performUpkeep)
		wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );
		weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );


198		uint256 btcPrice = priceAggregator.getPriceBTC();
199        uint256 ethPrice = priceAggregator.getPriceETH();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L172

```solidity
File: src/dao/DAO.sol

160			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
161			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );


221		if ( proposals.ballotIsApproved(ballotID ) )
222			{
223			Ballot memory ballot = proposals.ballotForID(ballotID);
224			_executeApproval( ballot );
225			}
226
		proposals.markBallotAsFinalized(ballotID);


254			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L160

```solidity
File: src/launch/InitialDistribution.sol

53		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
54
55   	// 52 million		Emissions
56		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );
57
58	    // 25 million		DAO Reserve Vesting Wallet
59		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );
60
61	    // 10 million		Initial Development Team Vesting Wallet
62		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );
63
64	    // 5 million		Airdrop Participants
65		salt.safeTransfer( address(airdrop), 5 * MILLION_ETHER );
66		airdrop.allowClaiming();
67
68		bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
69
70	    // 5 million		Liquidity Bootstrapping
71	    // 3 million		Staking Bootstrapping
72		salt.safeTransfer( address(saltRewards), 8 * MILLION_ETHER );
73		saltRewards.sendInitialSaltRewards(5 * MILLION_ETHER, poolIDs );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L53

```solidity
File: src/stable/Liquidizer.sol

94		usds.safeTransfer( address(usds), amountToBurn );
95		usds.burnTokensInContract();


123			dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
124			dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);

147			salt.safeTransfer(address(salt), saltBalance);
148			salt.burnTokensInContract();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L94

```solidity
File: src/staking/Staking.sol

118			salt.safeTransfer( address(salt), expeditedUnstakeFee );
119            salt.burnTokensInContract();


123		salt.safeTransfer( msg.sender, u.claimableSALT );


200		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
201        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
202        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L118

## [G-05] Consider using OZ EnumerateSet in place of nested mappings

```solidity
File: src/AccessManager.sol

26    mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L26

```solidity
File: src/pools/Pools.sol

49	mapping(address=>mapping(IERC20=>uint256)) private _userDeposits;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L49

```solidity
File: src/dao/Proposals.sol

51	mapping(uint256=>mapping(Vote=>uint256)) private _votesCastForBallot;

55	mapping(uint256=>mapping(address=>UserVote)) private _lastUserVoteForBallot;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L51

```solidity
File: src/staking/StakingRewards.sol

36	mapping(address=>mapping(bytes32=>UserShareInfo)) private _userShareInfo;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L36

## [G-06] Shorten the array rather than copying to a new one

```solidity
File: src/stable/CollateralAndLiquidity.sol

313		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);

317		address[] memory resizedLiquidatableUsers = new address[](count);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L313

```solidity
File: src/price_feed/CoreUniswapFeed.sol

52		uint32[] memory secondsAgo = new uint32[](2);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L52

```solidity
File: src/dao/DAO.sol

261			AddedReward[] memory addedRewards = new AddedReward[](2);

332		bytes32[] memory poolIDs = new bytes32[](2);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L261

```solidity
File: src/pools/PoolStats.sol

138		_calculatedProfits = new uint256[](poolIDs.length);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L138

```solidity
File: src/rewards/RewardsEmitter.sol

99		 	poolIDs = new bytes32[](1);

109		AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

144		uint256[] memory rewards = new uint256[]( pools.length );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L99

```solidity
File: src/rewards/SaltRewards.sol

49		AddedReward[] memory addedRewards = new AddedReward[](1);

63		AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

86		AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

98		AddedReward[] memory addedRewards = new AddedReward[](1);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L49

```solidity
File: src/staking/Staking.sol

160        Unstake[] memory unstakes = new Unstake[](end - start + 1);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L160

```solidity
File: src/staking/StakingRewards.sol

214		shares = new uint256[]( poolIDs.length );

224		rewards = new uint256[]( poolIDs.length );

258		rewards = new uint256[]( poolIDs.length );

275		shares = new uint256[]( poolIDs.length );

293		cooldowns = new uint256[]( poolIDs.length );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L214

## [G-07] Avoid unnecessary public variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

```solidity
File: src/AccessManager.sol

23	IDAO immutable public dao;

29    uint256 public geoVersion;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L23

```solidity
File: src/launch/Airdrop.sol

26	bool public claimingAllowed;

32	uint256 public saltAmountForEachUser;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L26

```solidity
File: src/launch/BootstrapBallot.sol

17    IExchangeConfig immutable public exchangeConfig;

18    IAirdrop immutable public airdrop;

19	uint256 immutable public completionTimestamp;

21	bool public ballotFinalized;
	bool public startExchangeApproved;

29	uint256 public startExchangeYes;

30	uint256 public startExchangeNo;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L17

```solidity
File: src/stable/CollateralAndLiquidity.sol


34    IStableConfig immutable public stableConfig;

35	IPriceAggregator immutable public priceAggregator;

36    IUSDS immutable public usds;

37	IERC20 immutable public wbtc;

38	IERC20 immutable public weth;

39	ILiquidizer immutable public liquidizer;

42	uint256 immutable public wbtcTenToTheDecimals;

43    uint256 immutable public wethTenToTheDecimals;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L34

```solidity
File: src/price_feed/CoreSaltyFeed.so


15    IPools immutable public pools;

17	IERC20 immutable public wbtc;

18	IERC20 immutable public weth;

19	IERC20 immutable public usds;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L15

```solidity
File: src/price_feed/CoreUniswapFeed.sol

21    IUniswapV3Pool immutable public UNISWAP_V3_WBTC_WETH;

22	  IUniswapV3Pool immutable public UNISWAP_V3_WETH_USDC;

24   IERC20 immutable public wbtc;

25   IERC20 immutable public weth;

26   IERC20 immutable public usdc;

28   bool immutable public wbtc_wethFlipped;

29    bool immutable public weth_usdcFlipped;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L21

```solidity
File: src/dao/DAO.sol

44	IPools immutable public pools;

45	IProposals immutable public proposals;

46	IExchangeConfig immutable public exchangeConfig;

47	IPoolsConfig immutable public poolsConfig;

48	IStakingConfig immutable public stakingConfig;

49	IRewardsConfig immutable public rewardsConfig;

50	IStableConfig immutable public stableConfig;

51	IDAOConfig immutable public daoConfig;

52	IPriceAggregator immutable public priceAggregator;

53	IRewardsEmitter immutable public liquidityRewardsEmitter;

54	ICollateralAndLiquidity immutable public collateralAndLiquidity;

55	ILiquidizer immutable public liquidizer;

57	ISalt immutable public salt;

58 IUSDS immutable public usds;

59	IERC20 immutable public dai;

63	string public websiteURL;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L44

```solidity
File: src/dao/DAOConfig.sol


24	uint256 public bootstrappingRewards = 200000 ether;

28    uint256 public percentPolRewardsBurned = 50;

40	uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000; // Default 10% of the total amount of SALT staked with a 1000x multiplier

45	uint256 public ballotMinimumDuration = 10 days;

49	uint256 public requiredProposalPercentStakeTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier

53	uint256 public maxPendingTokensForWhitelisting = 5;

57	uint256 public arbitrageProfitsPercentPOL = 20;

61	uint256 public upkeepRewardPercent = 5;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L24

```solidity
File: src/rewards/Emissions.sol


21    ISaltRewards immutable public saltRewards;

22	IExchangeConfig immutable public exchangeConfig;

23	IRewardsConfig immutable public rewardsConfig;

24	ISalt immutable public salt;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L21

```solidity
File: src/staking/Liquidity.sol

25	IPools immutable public pools;

    bytes32 immutable public collateralPoolID;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L25

```solidity
File: src/stable/Liquidizer.sol

31    IERC20 immutable public wbtc;

32    IERC20 immutable public weth;

33   IUSDS immutable public usds;

34    ISalt immutable public salt;

35    IERC20 immutable public dai;

37    IExchangeConfig public exchangeConfig;

38    IPoolsConfig immutable  public poolsConfig;

39    ICollateralAndLiquidity public collateralAndLiquidity;

41    IPools public pools;

42    IDAO public dao;

47	uint256 public usdsThatShouldBeBurned;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L31

```solidity
File: src/ManagedWallet.sol


20    address public mainWallet;

21    address public confirmationWallet;

24    address public proposedMainWallet;

25    address public proposedConfirmationWallet;

28    uint256 public activeTimelock;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L20

```solidity
File: src/pools/PoolStats.sol

16	IExchangeConfig immutable public exchangeConfig;

17	IPoolsConfig immutable public poolsConfig;

18	IERC20 immutable public _weth;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L16

```solidity
File: src/price_feed/PriceAggregator.sol

19	IPriceFeed public priceFeed1; // CoreUniswapFeed by default

21	IPriceFeed public priceFeed2; // CoreChainlinkFeed by default

22	IPriceFeed public priceFeed3; // CoreSaltyFeed by default

25	uint256 public priceFeedModificationCooldownExpiration;

31	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

36	uint256 public priceFeedModificationCooldown = 35 days;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L19

```solidity
File: src/dao/Proposals.sol

30    IStaking immutable public staking;

31    IExchangeConfig immutable public exchangeConfig;

32    IPoolsConfig immutable public poolsConfig;

33    IDAOConfig immutable public daoConfig;

34    ISalt immutable public salt;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L30

```solidity
File: src/rewards/RewardsConfig.sol

19	uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier

24	uint256 public emissionsWeeklyPercentTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier

28   uint256 public stakingRewardsPercent = 50;

34   uint256 public percentRewardsSaltUSDS = 10;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L19

```solidity
File: src/rewards/RewardsEmitter.

24	IStakingRewards immutable public stakingRewards;

25	IExchangeConfig immutable public exchangeConfig;

26	IPoolsConfig immutable public poolsConfig;

27	IRewardsConfig immutable public rewardsConfig;

28	ISalt immutable public salt;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L24

```solidity
File: src/rewards/SaltRewards.sol

19	IRewardsEmitter immutable public stakingRewardsEmitter;

20	IRewardsEmitter immutable public liquidityRewardsEmitter;

21	IExchangeConfig immutable public exchangeConfig;

22	IRewardsConfig immutable public rewardsConfig;

23	ISalt immutable public salt;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L19

```solidity
File: src/stable/StableConfig.sol

20    uint256 public rewardPercentForCallingLiquidation = 5;

24   uint256 public maxRewardValueForCallingLiquidation = 500 ether;

29   uint256 public minimumCollateralValueForBorrowing = 2500 ether;

34    uint256 public initialCollateralRatioPercent = 200;

39	uint256 public minimumCollateralRatioPercent = 110;

44	uint256 public percentArbitrageProfitsForStablePOL = 5;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L20

```solidity
File: src/staking/StakingConfig.sol

18	uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

22	uint256 public maxUnstakeWeeks = 52;

26	uint256 public minUnstakePercent = 20;

31	uint256 public modificationCooldown = 1 hours;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L18

```solidity
File: src/Upkeep.sol

47	IPools immutable public pools;

48	IExchangeConfig  immutable public exchangeConfig;

49	IPoolsConfig immutable public poolsConfig;

50	IDAOConfig immutable public daoConfig;

51	IStableConfig immutable public stableConfig;

52	IPriceAggregator immutable public priceAggregator;

53	ISaltRewards immutable public saltRewards;

54	ICollateralAndLiquidity immutable public collateralAndLiquidity;

55	IEmissions immutable public emissions;

56	IDAO immutable public dao;

58	IERC20  immutable public weth;

59	ISalt  immutable public salt;

60	IUSDS  immutable public usds;

61	IERC20  immutable public dai;

63	uint256 public lastUpkeepTimeEmissions;

64	uint256 public lastUpkeepTimeRewardsEmitters;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L47

## [G-08] Cache external calls outside of loop to avoid re-calling function on each iteration

```solidity
File: src/pools/PoolStats.sol

81		for( uint256 i = 0; i < poolIDs.length; i++ )
82			{
83			bytes32 poolID = poolIDs[i];
84			(IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81

```solidity
File: src/dao/Proposals.sol

423		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
424			{
425			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423

```solidity
File: src/rewards/RewardsEmitter.sol

60		for( uint256 i = 0; i < addedRewards.length; i++ )
61			{
62			AddedReward memory addedReward = addedRewards[i];
63			require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );// @audit EC
64
65			uint256 amountToAdd = addedReward.amountToAdd;
66			if ( amountToAdd != 0 )
67				{
68				// Update pendingRewards so the SALT can be distributed later
69				pendingRewards[ addedReward.poolID ] += amountToAdd;
70				sum += amountToAdd;
71				}
72			}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L63

```solidity
File: src/stable/CollateralAndLiquidity.sol

321			for ( uint256 i = startIndex; i <= endIndex; i++ )
322				{
323				address wallet = _walletsWithBorrowedUSDS.at(i);
324
325				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
326				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321

## [G-09] Use Constants instead of type(uintx).max

```solidity
File: src/dao/DAO.sol

90		salt.approve(address(collateralAndLiquidity), type(uint256).max);
91		usds.approve(address(collateralAndLiquidity), type(uint256).max);
92		dai.approve(address(collateralAndLiquidity), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L90

```solidity
File: src/stable/Liquidizer.sol

71		wbtc.approve( address(pools), type(uint256).max );
72		weth.approve( address(pools), type(uint256).max );
73		dai.approve( address(pools), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L71

```solidity
File: src/ManagedWallet.sol

37		activeTimelock = type(uint256).max;

68			activeTimelock = type(uint256).max; // effectively never

86		activeTimelock = type(uint256).max;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L37

```solidity
File: src/dao/Proposals.sol

165		require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L165

```solidity
File: src/rewards/RewardsEmitter.sol

50		salt.approve(address(stakingRewards), type(uint256).max);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L50

```solidity
File: src/rewards/SaltRewards.sol

41		salt.approve( address(stakingRewardsEmitter), type(uint256).max );
42		salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L41

```solidity
File: src/Upkeep.sol

91		weth.approve( address(pools), type(uint256).max );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L91