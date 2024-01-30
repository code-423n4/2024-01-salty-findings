## [G-01] Add unchecked{} for subtraction where the operands can't underflow becasue of previous IF-STATEMENT

**Issue Description**\
The subtraction operation `uint256 remainingSALT = claimedSALT - amountToSendToTeam;` does not have an unchecked block, even though underflow is not possible due to the previous check `if ( claimedSALT == 0 )` return. Adding an unchecked block helps avoid unnecessary checks.

**Proposed Optimization**\
Add an unchecked block around the subtraction:

```solidity
unchecked {
  uint256 remainingSALT = claimedSALT - amountToSendToTeam;
}
```

**Estimated Gas Savings**\
Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction. With many such subtractions occurring across interactions, the total gas savings could be significant.

**Attachments**

- **Code Snippets**

```solidity
File: src/dao/DAO.sol

337		if ( claimedSALT == 0 ) // @audit  Because of this check can't overflow   remainingSALT

345		uint256 remainingSALT = claimedSALT - amountToSendToTeam;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L337

```solidity
File: src/staking/Liquidity.sol

110		if ( addedAmountA < maxAmountA )
111			tokenA.safeTransfer( msg.sender, maxAmountA - addedAmountA );



113		if ( addedAmountB < maxAmountB )
114			tokenB.safeTransfer( msg.sender, maxAmountB - addedAmountB );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L110

```solidity
File: src/pools/PoolMath.sol

161		if ( maximumMSB > 80 )
162			shift = maximumMSB - 80;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L161

```solidity
File: src/price_feed/PriceAggregator.sol

101		if ( x > y )
102			return x - y;
103
104		return y - x;
105		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L101

```solidity
File: src/rewards/RewardsConfig.sol

40            if (rewardsEmitterDailyPercentTimes1000 < 2500)
41                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 + 250;
42            }
43        else
44            {
45            if (rewardsEmitterDailyPercentTimes1000 > 250)
46                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 - 250;
47            }


73            if (stakingRewardsPercent < 75)
74                stakingRewardsPercent = stakingRewardsPercent + 5;
75            }
76        else
77            {
78            if (stakingRewardsPercent > 25)
79                stakingRewardsPercent = stakingRewardsPercent - 5;
80            }
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L40

```solidity
File: src/staking/Staking.sol


179		return unstakesForUser( user, 0, unstakeIDs.length - 1 );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L179

```solidity
File: src/staking/StakingRewards.sol

132		if ( virtualRewardsToRemove < rewardsForAmount )
133			claimableRewards = rewardsForAmount - virtualRewardsToRemove;


248		if ( user.virtualRewards > rewardsShare )
249			return 0;

251		return rewardsShare - user.virtualRewards;
252		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L132

## [G-02] STATE VARIABLES can be packed into fewer STORAGE SLOTS

**Issue Description**\
The `lastUpkeepTimeEmissions` and `lastUpkeepTimeRewardsEmitters` state variables can be packed to save 1 storage slot.

**Proposed Optimization**\

```solidity
// File: src/Upkeep.sol

struct UpkeepTimes {
  uint48 lastUpkeepTimeEmissions;
  uint48 lastUpkeepTimeRewardsEmitters;
}

UpkeepTimes public upkeepTimes;

constructor() {
  upkeepTimes.lastUpkeepTimeEmissions = uint48(block.timestamp);
  upkeepTimes.lastUpkeepTimeRewardsEmitters = uint48(block.timestamp);
}
```

**Estimated Gas Savings**\
Packing the two uint256 state variables into a struct saves 1 storage slot, resulting in reduced fees for deployment, external calls, and state updates.

**Attachments**

- **Code Snippets**

```solidity
File: src/Upkeep.sol

63	uint256 public lastUpkeepTimeEmissions;
64	uint256 public lastUpkeepTimeRewardsEmitters;


		lastUpkeepTimeEmissions = block.timestamp;
		lastUpkeepTimeRewardsEmitters = block.timestamp;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L63-L63

## [G-03] Functions Guaranteed to revert when called by normal users can be markd payable

### These are missed from bot

**Issue Description**\
By default, Solidity checks that the call value is 0 for non-payable functions, incurring unnecessary gas costs for admin functions that do not expect/process ether.

**Proposed Optimization**\
For admin functions that only restrict access and do not process Ether, declare them as payable to avoid call value checks.

**Estimated Gas Savings**\
As noted, making an admin function payable saves gas by removing the unnecessary call value check opcodes.

This reduces contract size and deployment costs. Each removed opcode saves non-trivial amounts of gas.

For contracts with many access-restricted admin functions, this optimization can provide meaningful cumulative savings, especially as usage scales up over time.

**Attachments**

- **Code Snippets**

```solidity
File:  src/AccessManager.sol

41    function excludedCountriesUpdated() external
42    	{
43    	require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L41

```solidity
File: src/launch/Airdrop.sol

46    function authorizeWallet( address wallet ) external
47    	{
48    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
49    	require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );


56    function allowClaiming() external
57    	{
58    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L46

```solidity
File: src/dao/DAO.sol

295	function withdrawArbitrageProfits( IERC20 weth ) external returns (uint256 withdrawnAmount)
296		{
297		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

316	function formPOL( IERC20 tokenA, IERC20 tokenB, uint256 amountA, uint256 amountB ) external
317		{
318		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );



327	function processRewardsFromPOL() external
328		{
329		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );


360	function withdrawPOL( IERC20 tokenA, IERC20 tokenB, uint256 percentToLiquidate ) external
361		{
362		require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L295

```solidity
File: src/rewards/Emissions.sol

40	function performUpkeep(uint256 timeSinceLastUpkeep) external
41		{
42		require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L40

```solidity
File: src/launch/InitialDistribution.sol

50    function distributionApproved() external
51    	{
52    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L50

```solidity
File: src/stable/Liquidizer.sol

81	function incrementBurnableUSDS( uint256 usdsToBurn ) external
82		{
83		require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );


132	function performUpkeep() external
133		{
134		require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L81

```solidity
File: src/ManagedWallet.sol

42	function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
43		{
44		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L42

```solidity
File: src/pools/Pools.sol

70	function startExchangeApproved() external nonReentrant
71		{
72    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );



140	function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
141		{
142		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );


170	function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
171		{
172		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L70

```solidity
File: src/pools/PoolStats.sol

47	function clearProfitsForPools() external
48		{
49		require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L47

```solidity
File: src/dao/Proposals.sol

122	function createConfirmationProposal( string calldata ballotName, BallotType ballotType, address address1, string calldata string1, string calldata description ) external returns (uint256 ballotID)
123		{
124		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

130	function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
131		{
132		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L122

```solidity
File: src/rewards/RewardsEmitter.sol

82	function performUpkeep( uint256 timeSinceLastUpkeep ) external
83		{
84		require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L82

```solidity
File: src/rewards/SaltRewards.sol

106	function sendInitialSaltRewards( uint256 liquidityBootstrapAmount, bytes32[] calldata poolIDs ) external
107		{
108		require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );


117	function performUpkeep( bytes32[] calldata poolIDs, uint256[] calldata profitsForPools ) external
118		{
119		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L106

## [G-04] State variables should be cached in stack variables rather than re-reading them from storage

### These are missed from bots

```solidity
File: src/launch/Airdrop.sol

59    	require( ! claimingAllowed, "Claiming is already allowed" );

69    	claimingAllowed = true;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L59

```solidity
File: src/launch/BootstrapBallot.sol

71		require( ! ballotFinalized, "Ballot has already been finalized" );

84		ballotFinalized = true;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L71

```solidity
File: src/ExchangeConfig.sol

51		require( address(dao) == address(0), "setContracts can only be called once" );

53		dao = _dao;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51

```solidity
File: src/stable/Liquidizer.sol

84		usdsThatShouldBeBurned += usdsToBurn;
87		emit incrementedBurnableUSDS(usdsThatShouldBeBurned);

119			usdsThatShouldBeBurned -= usdsBalance;
123			dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
124			dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L66

```solidity
File: src/ManagedWallet.sol

77		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
78		confirmationWallet = proposedConfirmationWallet;
79		emit WalletChange(mainWallet, confirmationWallet);
80		activeTimelock = type(uint256).max;
81		proposedMainWallet = address(0);
82		proposedConfirmationWallet = address(0);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L49

```solidity
File: src/price_feed/PriceAggregator.sol

50		if ( block.timestamp < priceFeedModificationCooldownExpiration )
69		priceFeedModificationCooldownExpiration = block.timestamp + priceFeedModificationCooldown;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L50

## [G-05] Replace STATE VARIABLE reads and writes within loops with STACK VARIABLE reads and writes

**Issue Description**\
Accessing state variables within loops incurs additional gas costs due to the storage and memory access operations involved for each read or write. These costs accumulate quickly, particularly in loops with many iterations.

**Proposed Optimization**\
Replace reads and writes of state variables within loops with reads and writes of local stack variables instead. Stack variables do not incur the additional costs associated with storage/memory accesses.

**Estimated Gas Savings**\
Gas costs for each state variable access within a loop would be avoided. For loops with a large number of iterations, this could result in significant overall gas savings compared to accessing state variables directly. The exact savings would depend on specifics like the number of accessed variables and loop iterations.

**Attachments**

- **Code Snippets**

```solidity
File: src/stable/CollateralAndLiquidity.sol

321			for ( uint256 i = startIndex; i <= endIndex; i++ )
322				{
323				address wallet = _walletsWithBorrowedUSDS.at(i);//@audit _walletsWithBorrowedUSDS   ST
324
325				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
326				// @audit usdsBorrowedByUsers   ST
327				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
328
329				// Determine minCollateral in terms of minCollateralValue
330			uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
331
332				// Make sure the user has at least minCollateral
333				if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )
334					liquidatableUsers[count++] = wallet;
335				}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321-L334

```solidity
File: src/pools/PoolStats.sol

53		for( uint256 i = 0; i < poolIDs.length; i++ )
54			_arbitrageProfits[ poolIDs[i] ] = 0;
55		}



106		for( uint256 i = 0; i < poolIDs.length; i++ )
107			{
108			// references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
109			bytes32 poolID = poolIDs[i];
110
111			// Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
112			uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
113			if ( arbitrageProfit > 0 )
114				{
115				ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L53

```solidity
File: src/dao/Proposals.sol

423		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
424			{
425			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
426			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
427			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];
428
429			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
430			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
431			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
432				{
433				bestID = ballotID;
434				mostYes = yesTotal;
435			}
436			}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423-L436

```solidity
File: src/rewards/RewardsEmitter.sol

116		for( uint256 i = 0; i < poolIDs.length; i++ )
117			{
118			bytes32 poolID = poolIDs[i];
119
120			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
121			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;
122
123			// Reduce the pending rewards so they are not sent again
124			if ( amountToAddForPool != 0 )
125				{
126				pendingRewards[poolID] -= amountToAddForPool;




146		for( uint256 i = 0; i < rewards.length; i++ )
147			rewards[i] = pendingRewards[ pools[i] ];
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L116

```solidity
File: src/staking/StakingRewards.sol


216		for( uint256 i = 0; i < shares.length; i++ )
217			shares[i] = totalShares[ poolIDs[i] ];
218		}



226		for( uint256 i = 0; i < rewards.length; i++ )
227			rewards[i] = totalRewards[ poolIDs[i] ];
228		}


277		for( uint256 i = 0; i < shares.length; i++ )
278			shares[i] = _userShareInfo[wallet][ poolIDs[i] ].userShare;
279		}



296		for( uint256 i = 0; i < cooldowns.length; i++ )
297			{
298			uint256 cooldownExpiration = userInfo[ poolIDs[i] ].cooldownExpiration;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L216

## [G-06] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042


contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734


contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message
with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks
the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: src/AccessManager.sol

43    	require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

63    	require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol

```solidity
File: src/launch/Airdrop.sol

48    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

49    	require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );

58    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );

59    	require( ! claimingAllowed, "Claiming is already allowed" );

60		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

76    	require( claimingAllowed, "Claiming is not allowed yet" );

77    	require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );

78    	require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol

```solidity
File: src/launch/BootstrapBallot.sol

36		require( ballotDuration > 0, "ballotDuration cannot be zero" );

50		require( ! hasVoted[msg.sender], "User already voted" );

54		require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );

71		require( ! ballotFinalized, "Ballot has already been finalized" );

72		require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L36

```solidity
File: src/stable/CollateralAndLiquidity.sol

83		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

84		require( collateralToWithdraw <= maxWithdrawableCollateral(msg.sender), "Excessive collateralToWithdraw" );

97		require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

98		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

99		require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );

117		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

118		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );

119		require( amountRepaid > 0, "Cannot repay zero amount" );

142		require( wallet != msg.sender, "Cannot liquidate self" );

145		require( canUserBeLiquidated(wallet), "User cannot be liquidated" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol

```solidity
File: src/dao/DAO.sol

247			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

251			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

281		require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );

297		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

318		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329		require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

362		require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol

```solidity
File: src/rewards/Emissions.sol

42		require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol

```solidity
File: src/ExchangeConfig.sol

51		require( address(dao) == address(0), "setContracts can only be called once" );

64		require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol

```solidity
File: src/launch/InitialDistribution.sol

52    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );

53		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol

```solidity
File: src/staking/Liquidity.sol

42		require(block.timestamp <= deadline, "TX EXPIRED");

85		require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

124		require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

148		require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be deposited via Liquidity.depositLiquidityAndIncreaseShare" );

159		require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be withdrawn via Liquidity.withdrawLiquidityAndClaim" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/

```solidity
File: src/stable/Liquidizer.sol

83		require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

134		require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol

```solidity
File: src/ManagedWallet.sol

44		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

45		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );

46		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );

49		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );

61    	require( msg.sender == confirmationWallet, "Invalid sender" );

76		require( msg.sender == proposedMainWallet, "Invalid sender" );

77		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol

```solidity
File: src/pools/Pools.sol

72    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

83		require(block.timestamp <= deadline, "TX EXPIRED");


142		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

143		require( exchangeIsLive, "The exchange is not yet live" );

144		require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );

146		require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );

147		require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );

158		require( addedLiquidity >= minLiquidityReceived, "Too little liquidity received" );

172		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

173		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

187        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

193		require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

207        require( amount > PoolUtils.DUST, "Deposit amount too small");

221    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

222        require( amount > PoolUtils.DUST, "Withdraw amount too small");

243        require((reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves before swap");

271        require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap");

353		require( swapAmountOut >= minAmountOut, "Insufficient resulting token amount" );

371    	require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol

```solidity
File: src/pools/PoolsConfig.sol

47		require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );

48		require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");

136		require(address(pair.tokenA) != address(0) && address(pair.tokenB) != address(0), "This poolID does not exist");
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol

```solidity
File: src/pools/PoolStats.sol

49		require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol

```solidity
File: src/price_feed/PriceAggregator.sol

39		require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol

```solidity
File: src/dao/Proposals.sol

83		require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

92			require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

95			require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );

98			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );

102		require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );

103		require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );

124		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

164		require( address(token) != address(0), "token cannot be address(0)" );

165		require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision

167		require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );

168		require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );

169		require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );

182		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );

183		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );

184		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );

185		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );

186		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );

187		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

198		require( wallet != address(0), "Cannot send SALT to address(0)" );

203		require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

215		require( contractAddress != address(0), "Contract address cannot be address(0)" );

224		require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

233		require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

242		require( newAddress != address(0), "Proposed address cannot be address(0)" );

251		require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );

264		require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

268			require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );

277		require( userVotingPower > 0, "Staked SALT required to vote" );

321		require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol

```solidity
File: src/rewards/RewardsEmitter.sol

63			require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );

84		require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol

```solidity
File: src/rewards/SaltRewards.sol

60		require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

108		require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );

119		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );

120		require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol

```solidity
File: src/staking/Staking.sol

43		require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

62		require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" );


88		require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" );

89		require( block.timestamp < u.completionTime, "Unstakes that have already completed cannot be cancelled" );

90		require( msg.sender == u.wallet, "Sender is not the original staker" );

104		require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" );

105		require( block.timestamp >= u.completionTime, "Unstake has not completed yet" );

106		require( msg.sender == u.wallet, "Sender is not the original staker" );

132		require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );

153        require(end >= start, "Invalid range: end cannot be less than start");

157        require(userUnstakes.length > end, "Invalid range: end is out of bounds");

158        require(start < userUnstakes.length, "Invalid range: start is out of bounds");

204		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );

205		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol

```solidity
File: src/staking/StakingRewards.sol

58        require(poolsConfig.isWhitelisted(poolID), "Invalid pool");

59        require(increaseShareAmount != 0, "Cannot increase zero share");

67                require(block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire");

101        require(decreaseShareAmount != 0, "Cannot decrease zero share");

104        require(decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share");

110                require(block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire");

189            require(poolsConfig.isWhitelisted(poolID), "Invalid pool");
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol

```solidity
File: src/stable/USDS.sol

42		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );

43		require( amount > 0, "Cannot mint zero USDS" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol

## [G-07] Multiple ADDRESS/ID mapping can be combined into a single mapping of an ADDRESS/ID to a STRUCT

**Issue Description**\
The `_arbitrageProfits` and `_arbitrageIndicies` mappings can be combined into a single mapping to reduce storage usage and transaction costs.

**Proposed Optimization**

```diff

-	mapping(bytes32=>uint256) public _arbitrageProfits;
-	mapping(bytes32=>ArbitrageIndicies) public _arbitrageIndicies;



+      struct _arbitrageProfitsIndicies{
+        uint256 _arbitrageProfits;
+        ArbitrageIndicies _arbitrageIndicies;
+    }
+
+	mapping(bytes32 => _arbitrageProfitsIndicies) public arbitrage; // True if collateral locker was created by this factory, otherwise false
```

**Estimated Gas Savings**\
Eliminates overhead of storing and retrieving separate keys ~42 gas saved per access by not recomputing keccak256 hash

**Attachments**

- **Code Snippets**

```solidity
File: src/pools/PoolStats.sol

20	// poolID(arbToken2, arbToken3) => arbitrage profits contributed since the last performUpkeep
21	mapping(bytes32=>uint256) public _arbitrageProfits;


23	// Maps poolID(arbToken2, arbToken3) => the indicies (within the whitelistedPools array) of the pools involved in WETH->arbToken2->arbToken3->WETH
24	mapping(bytes32=>ArbitrageIndicies) public _arbitrageIndicies;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L20

```solidity
File: src/staking/StakingRewards.sol

38    // A mapping that stores the total pending SALT rewards for each poolID.
39    mapping(bytes32=>uint256) public totalRewards;


41    // A mapping that stores the total shares for each poolID.
42    mapping(bytes32=>uint256) public totalShares;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L38

## [G-08] Access MAPPINGS directly rather than using accessor functions

**Issue Description**\
When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

**Proposed Optimization**\
Access mappings directly inside functions instead of through accessor functions whenever possible to remove the extra layer of indirection.

**Estimated Gas Savings**\
Reading a mapping value directly saves approximately `600` gas vs using an accessor function.

**Attachments**

- **Code Snippets**

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

## [G-09] Use ASSEMBLY to validate msg.sender

**Issue Description**\
Validation of msg.sender is commonly used but can be costly due to the calculation.

**Proposed Optimization**\
Use inline assembly to directly compare the msg.sender value rather than hashing.

```solidity
function validateSender(address expected) public {
  assembly {
    let actual := caller()
    jumpi(invalid, ne(actual, expected))
  }
}
```

**Estimated Gas Savings**\
Eliminates the need for the operation, saving approximately 30 gas per validation

**Attachments**

- **Code Snippets**

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

## [G-10] Consider using OZ EnumerateSet in place of nested mappings

**Issue Description**\
Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).

A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.

OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

**Proposed Optimization**\
The nested mapping `mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;` can be inefficient due to double hashing.

**Estimated Gas Savings**\
EnumerableSet avoids the cost of double hashing and provides more efficient set operations than a nested mapping. Exact savings would need to be measured.

**Attachments**

- **Code Snippets**

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

## [G-11] Expensive operation inside a for loop

**Proposed Optimization**\
Move expensive external calls out of the loop by:

- Computing splits & recipients arrays first
- Making a single bulk ETH transfer to the ERC20 contract
- Having ERC20 contract distribute tokens to recipients

**Estimated Gas Savings**\
Reduces external calls from O(n) to O(1). Can save >10k gas for larger creator lists.

**Attachments**

- **Code Snippets**

```solidity
File: src/stable/CollateralAndLiquidity.sol

321			for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
				address wallet = _walletsWithBorrowedUSDS.at(i);

				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;

				// Determine minCollateral in terms of minCollateralValue
				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;

				// Make sure the user has at least minCollateral
				if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )
					liquidatableUsers[count++] = wallet;
				}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321-L334

```solidity
File: src/staking/StakingRewards.sol

185        for (uint256 i = 0; i < addedRewards.length; i++) {
            AddedReward memory addedReward = addedRewards[i];

            bytes32 poolID = addedReward.poolID;
            require(poolsConfig.isWhitelisted(poolID), "Invalid pool");

            uint256 amountToAdd = addedReward.amountToAdd;

            totalRewards[poolID] += amountToAdd;
            sum = sum + amountToAdd;

            emit SaltRewardsAdded(poolID, amountToAdd);
        }
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185-L197

```solidity
File: src/pools/PoolStats.sol

81		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];
			(IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);

			// The middle two tokens can never be WETH in a valid arbitrage path as the path is WETH->arbToken2->arbToken3->WETH.
			if ( (arbToken2 != _weth) && (arbToken3 != _weth) )
				{
				uint64 poolIndex1 = _poolIndex( _weth, arbToken2, poolIDs );
				uint64 poolIndex2 = _poolIndex( arbToken2, arbToken3, poolIDs );
				uint64 poolIndex3 = _poolIndex( arbToken3, _weth, poolIDs );

				// Check if the indicies in storage have the correct values - and if not then update them
				ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
				if ( ( poolIndex1 != indicies.index1 ) || ( poolIndex2 != indicies.index2 ) || ( poolIndex3 != indicies.index3 ) )
					_arbitrageIndicies[poolID] = ArbitrageIndicies(poolIndex1, poolIndex2, poolIndex3);
				}
			}




106		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			// references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
			bytes32 poolID = poolIDs[i];

			// Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
			uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
			if ( arbitrageProfit > 0 )
				{
				ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];

				if ( indicies.index1 != INVALID_POOL_ID )
					_calculatedProfits[indicies.index1] += arbitrageProfit;

				if ( indicies.index2 != INVALID_POOL_ID )
					_calculatedProfits[indicies.index2] += arbitrageProfit;

				if ( indicies.index3 != INVALID_POOL_ID )
					_calculatedProfits[indicies.index3] += arbitrageProfit;
				}
			}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81-L98

```solidity
File: src/dao/Proposals.sol

423		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];

			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
				{
				bestID = ballotID;
				mostYes = yesTotal;
				}
			}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423

```solidity
File: src/rewards/RewardsEmitter.sol

116		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];

			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;

			// Reduce the pending rewards so they are not sent again
			if ( amountToAddForPool != 0 )
				{
				pendingRewards[poolID] -= amountToAddForPool;

				sum += amountToAddForPool;
				}

			// Specify the rewards that will be added for the specific pool
			addedRewards[i] = AddedReward( poolID, amountToAddForPool );
			}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L116-L133

## [G-12] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.

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

## [G-13] Shorten the array rather than copying to a new one

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

## [G-14] Make 3 Event parameters indexed when possible

### This is found by bot `[N-22]` Use indexed for event parameters but if there are less than 3 paramteer we need to make all indexed

**Issue Description**\
It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

**Proposed Optimization**\
Review all events and make the first 3 non-address parameters indexed if possible, to optimize for gas efficiency of log
encoding. If an event has fewer than 3 parameters, make them all indexed.

**Estimated Gas Savings**\
Each indexed parameter saves approximately 200 gas vs an unindexed parameter by reducing log encoding size. With many events,
this optimization can add up to significant gas savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/AccessManager.sol

21	event AccessGranted(address indexed wallet, uint256 geoVersion);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L21

```solidity
File: src/launch/BootstrapBallot.sol

15	event BallotFinalized(bool startExchange);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L15

```solidity
File: src/stable/CollateralAndLiquidity.sol

24    event CollateralDeposited(address indexed depositor, uint256 amountWBTC, uint256 amountWETH, uint256 liquidity);
25    event CollateralWithdrawn(address indexed withdrawer, uint256 collateralWithdrawn, uint256 reclaimedWBTC, uint256 reclaimedWETH);
26    event BorrowedUSDS(address indexed borrower, uint256 amountBorrowed);
27    event RepaidUSDS(address indexed repayer, uint256 amountRepaid);
28    event Liquidation(address indexed liquidator, address indexed liquidatee, uint256 reclaimedWBTC, uint256 reclaimedWETH, uint256 originallyBorrowedUSDS);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L24

```solidity
File: src/dao/DAO.sol

25	event BallotFinalized(uint256 indexed ballotID, Vote winningVote);

31    event ArbitrageProfitsWithdrawn(address indexed upkeepContract, IERC20 indexed weth, uint256 withdrawnAmount);

32    event SaltSent(address indexed to, uint256 amount);
36    event POLFormed(IERC20 indexed tokenA, IERC20 indexed tokenB, uint256 amountA, uint256 amountB);

38    event POLWithdrawn(IERC20 indexed tokenA, IERC20 indexed tokenB, uint256 withdrawnA, uint256 withdrawnB);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L25

```solidity
File: src/pools/Pools.sol

25	event LiquidityAdded(IERC20 indexed tokenA, IERC20 indexed tokenB, uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity);
	event LiquidityRemoved(IERC20 indexed tokenA, IERC20 indexed tokenB, uint256 reclaimedA, uint256 reclaimedB, uint256 removedLiquidity);
	event TokenDeposit(address indexed user, IERC20 indexed token, uint256 amount);
	event TokenWithdrawal(address indexed user, IERC20 indexed token, uint256 amount);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L25

```solidity
File: src/dao/Proposals.sol

23    event ProposalCreated(uint256 indexed ballotID, BallotType ballotType, string ballotName);

25    event VoteCast(address indexed voter, uint256 indexed ballotID, Vote vote, uint256 votingPower);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L23

```solidity
File: src/staking/Staking.sol

17	event SALTStaked(address indexed user, uint256 amountStaked);
18	event UnstakeInitiated(address indexed user, uint256 indexed unstakeID, uint256 amountUnstaked, uint256 claimableSALT, uint256 numWeeks);
19	event UnstakeCancelled(address indexed user, uint256 indexed unstakeID);
20	event SALTRecovered(address indexed user, uint256 indexed unstakeID, uint256 saltRecovered, uint256 expeditedUnstakeFee);
21	event XSALTTransferredFromAirdrop(address indexed toUser, uint256 amountTransferred);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L17

```solidity
File: src/staking/StakingRewards.sol

23	event UserShareIncreased(address indexed wallet, bytes32 indexed poolID, uint256 amountIncreased);
24	event UserShareDecreased(address indexed wallet, bytes32 indexed poolID, uint256 amountDecreased, uint256 claimedRewards);
25	event RewardsClaimed(address indexed wallet, uint256 claimedRewards);
26	event SaltRewardsAdded(bytes32 indexed poolID, uint256 amountAdded);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L23

## [G-15] Use Constants instead of type(uintx).max

**Issue Description**\
Using type(uint256).max to represent the maximum approvable amount for ERC20 tokens uses more gas in the distribution process
and also for each transaction than using a constant.

**Proposed Optimization**\
Define a constant such as MAX_UINT256 = type(uint256).max and use that constant instead.

**Estimated Gas Savings**\
Using a constant avoids the overhead of calling the type(uint256) method each time. This could save ~100 gas per transaction.
For contracts with many transactions, this can add up to significant savings over time.

**Attachments**

- **Code Snippets**

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

## [G-16] Avoid unnecessary public variables

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
File: src/ExchangeConfig.sol

17	ISalt immutable public salt;

18	IERC20 immutable public wbtc;

19	IERC20 immutable public weth;

21	IERC20 immutable public dai;

22	IUSDS immutable public usds;

23	IManagedWallet immutable public managedTeamWallet;

25	IDAO public dao; // can only be set once

26	IUpkeep public upkeep; // can only be set once

27	IInitialDistribution public initialDistribution; // can only be set once

28	IAirdrop public airdrop; // can only be set once

30	// Gradually distribute SALT to the teamWallet and DAO over 10 years

31	VestingWallet public teamVestingWallet;		// can only be set once

32	VestingWallet public daoVestingWallet;		// can only be set once

34	IAccessManager public accessManager;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L17

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

## [G-17] Cache external calls outside of loop to avoid re-calling function on each iteration

**Issue Description**\
Calling external functions like STATICCALL inside a loop incurs a gas cost for each iteration that can be optimized.

**Proposed Optimization**\
Cache results of external calls made prior to loop in local variables, and reference the cache inside the loop body.

**Estimated Gas Savings**\
Avoiding a re-call of an external function saves 100 gas per loop iteration. Significant savings for medium/large loops.

```solidity
// Without caching:

for (uint i = 0; i < 10; i++) {
  // Call external func
}

// With caching:

uint cache = externalFunc();

for (uint i = 0; i < 10; i++) {
  // Use cache
}
```

**Attachments**

- **Code Snippets**

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

## [G-18] Pre-increment and pre-decrement are cheaper than +1 ,-1

**Issue Description**\
The code is using +1 and -1 to increment and decrement variables.

**Proposed Optimization**\
Replace instances of +1 with ++ for pre-decrement, and -1 with -- for pre-increment.

**Estimated Gas Savings**\
pre-increment/decrement can save 2 gas per operation compared to adding/subtracting 1.

**Attachments**

- **Code Snippets**

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
