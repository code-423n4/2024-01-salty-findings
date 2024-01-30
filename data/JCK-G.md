
| Number | 	Issue | Instances |
|--------|--------|-----------|
|[G-01]| Cache external calls outside of loop to avoid re-calling function on each iteration   | 1 |
|[G-02]| Use assembly to validate msg.sender | 43 |
|[G-03]| State variables should be cached in stack variables rather than re-reading them from storage   | 30 |
|[G-04]| Use assembly to reuse memory space when making more than one external call   | 15 |
|[G-05]| Using delete statement can save gas | 2 |
|[G-06]| Use hardcoded address instead of address(this)   | 20 |
|[G-07]| Emit local variables instead of state variable (Save ~100 Gas)   | 14 |
|[G-08]| Avoid calling the same internal function multiple times   | 1 |
|[G-09]| Using assembly to revert with an error message   | 131 |
|[G-10]| Shorten arrays with inline assembly   | 19 |
|[G-11]| Don’t make variables public unless necessary   | 50 |
|[G-12]| When using storage instead of memory, we should cache any fields that need to be re-read in stack variables   | 7 |
|[G-13]| Use constants for variables that don’t change (Save a storage SLOT: 2200 Gas)   | 2 |
|[G-14]| Validate all parameters first before performing any other operations if there’s no side effects   | 6 |
|[G-15]| Use the already cached variable instead of reading from storage (Save 1 SLOAD: 100 Gas)   | 2 |
|[G-16]| Use uint256(1)/uint256(2) instead for true and false boolean states   | 12 |
|[G-17]| Add unchecked {} for subtractions where the operands can’t underflow because of previous checks   | 31 |
|[G-18]| Avoid contract existence checks by using low level calls   | 7 |

## [G-01] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

Cache minimumCollateralRatioPercent() external function outside of loop to save 1 STATICCALL per loop iteration

```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

321   for ( uint256 i = startIndex; i <= endIndex; i++ )
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


## [G-02] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: blob/main/src/dao/Proposals.sol

86    if ( msg.sender != address(exchangeConfig.dao() ) )

98    require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );

124   require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132   require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L86


```solidity
file: blob/main/src/staking/StakingRewards.sol

65   if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown

105  if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L65

```solidity
file: blob/main/src/Upkeep.sol

97   require(msg.sender == address(this), "Only callable from within the same contract");

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97 


```solidity
file: blob/main/src/launch/BootstrapBallot.sol

50   require( ! hasVoted[msg.sender], "User already voted" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L50

```solidity
file: blob/main/src/launch/InitialDistribution.sol

52   require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52


```solidity
file: blob/main/src/pools/Pools.sol

72   require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

142   require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

172   require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

221   require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L142

```solidity
file:  blob/main/src/pools/PoolStats.sol

49    require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L49


```solidity
file:  blob/main/src/rewards/Emissions.sol

42    require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42

```solidity
file: blob/main/src/rewards/RewardsEmitter.sol

84   require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L84

```solidity
file: blob/main/src/launch/Airdrop.sol

48   require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

58   require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );

77   require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );

78   require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48


```solidity
file: blob/main/src/rewards/SaltRewards.sol

108   require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );

119    require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108


```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

83    require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

84	  require( collateralToWithdraw <= maxWithdrawableCollateral(msg.sender), "Excessive collateralToWithdraw" );

97    require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

98	  require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

99	  require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );

117   require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

118	  require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );

142   require( wallet != msg.sender, "Cannot liquidate self" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L83


```solidity
file: blob/main/src/stable/Liquidizer.sol

83   require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

134   require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83

```solidity
file: blob/main/src/staking/Staking.sol

90   require( msg.sender == u.wallet, "Sender is not the original staker" );

106  require( msg.sender == u.wallet, "Sender is not the original staker" );

132  require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L90

```solidity
file: blob/main/src/staking/Liquidity.sol

124   require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L124

```solidity
file: blob/main/src/ManagedWallet.sol

44   require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

61   require( msg.sender == confirmationWallet, "Invalid sender" );

76   require( msg.sender == proposedMainWallet, "Invalid sender" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44

```solidity
file:  blob/main/src/dao/DAO.sol

297    require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

329   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297

```solidity
file: blob/main/src/AccessManager.sol

43   require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

63   require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43

## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

Issue Description - In the changeWallets() function, the state variable proposedMainWallet is read directly from storage on line 76,80,87 and the variable confirmationWallet in line 81 and 83 

Proposed Optimization - Cache the current value of proposedMainWallet in a local stack variable before updating storage. This avoids an unnecessary re-read of the storage.

```solidity
file: blob/main/src/ManagedWallet.sol

73  function changeWallets() external
		{
		// proposedMainWallet calls the function - to make sure it is a valid address.
		require( msg.sender == proposedMainWallet, "Invalid sender" );
		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

		// Set the wallets
		mainWallet = proposedMainWallet;
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet);

		// Reset
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L73-89

Issue Description - In the proposeWallets() function, the state variable proposedMainWallet is read directly from storage on line 49,51 and 54 when emitting the event ChangeUnwrapFee. This is inefficient, as it incurs an extra storage read operation that is unnecessary.

Proposed Optimization - Cache the current value of proposedMainWallet in a local stack variable before emitting the event and updating storage. This avoids an unnecessary re-read of the storage.

```solidity
file: blob/main/src/ManagedWallet.sol
 
42    function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
		{
		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );

		// Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );

		proposedMainWallet = _proposedMainWallet;
		proposedConfirmationWallet = _proposedConfirmationWallet;

		emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L42-L55


Issue Description - In the step6() function, the state variable lastUpkeepTimeEmissions is read directly from storage on line 186 and 189 the variable confirmationWallet in line 81 and 83 

Proposed Optimization - Cache the current value of lastUpkeepTimeEmissions in a local stack variable before updating storage. This avoids an unnecessary re-read of the storage.

and also the step8() function the state variable lastUpkeepTimeRewardsEmitters

```solidity
file:  blob/main/src/Upkeep.sol

184  	function step6() public onlySameContract
		{
		uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions;
		emissions.performUpkeep(timeSinceLastUpkeep);

		lastUpkeepTimeEmissions = block.timestamp;
		}

205   function step8() public onlySameContract
		{
		uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeRewardsEmitters;

		saltRewards.stakingRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
		saltRewards.liquidityRewardsEmitter().performUpkeep(timeSinceLastUpkeep);

		lastUpkeepTimeRewardsEmitters = block.timestamp;
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L184-L190

cache state variable in the following function 

```solidity
file: blob/main/src/dao/DAOConfig.sol

64  function changeBootstrappingRewards(bool increase) external onlyOwner
		{
        if (increase)
        	{
            if (bootstrappingRewards < 500000 ether)
                bootstrappingRewards += 50000 ether;
            }
       	 else
       	 	{
            if (bootstrappingRewards > 50000 ether)
                bootstrappingRewards -= 50000 ether;
	        }

		emit BootstrappingRewardsChanged(bootstrappingRewards);
    	}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L64-L78


```solidity
file:  blob/main/src/dao/DAOConfig.sol

81  function changePercentPolRewardsBurned(bool increase) external onlyOwner
		{
		if (increase)
			{
			if (percentPolRewardsBurned < 75)
				percentPolRewardsBurned += 5;
			}
		else
			{
			if (percentPolRewardsBurned > 25)
				percentPolRewardsBurned -= 5;
			}

		emit PercentPolRewardsBurnedChanged(percentPolRewardsBurned);
		}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L81-L95

```solidity
file: blob/main/src/dao/DAOConfig.sol
 
98  function changeBaseBallotQuorumPercent(bool increase) external onlyOwner
		{
		if (increase)
			{
			if (baseBallotQuorumPercentTimes1000 < 20 * 1000)
				baseBallotQuorumPercentTimes1000 += 1000;
			}
		else
			{
			if (baseBallotQuorumPercentTimes1000 > 5 * 1000 )
				baseBallotQuorumPercentTimes1000 -= 1000;
			}

		emit BaseBallotQuorumPercentChanged(baseBallotQuorumPercentTimes1000);
		}

```


https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L98-L112

```solidity
file: blob/main/src/dao/DAOConfig.sol

115  function changeBallotDuration(bool increase) external onlyOwner
    	{
        if (increase)
        	{
            if (ballotMinimumDuration < 14 days)
                ballotMinimumDuration += 1 days;
        	}
        else
        	{
            if (ballotMinimumDuration > 3 days)
                ballotMinimumDuration -= 1 days;
        	}

		emit BallotDurationChanged(ballotMinimumDuration);
    	}
```


https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L115-L129

```solidity
file: blob/main/src/dao/DAOConfig.sol

132   function changeRequiredProposalPercentStake(bool increase) external onlyOwner
		{
		if (increase)
			{
			if (requiredProposalPercentStakeTimes1000 < 2000) // Maximum 2%
				requiredProposalPercentStakeTimes1000 += 100; // Increase by 0.10%
			}
		else
			{
			if (requiredProposalPercentStakeTimes1000 > 100) // Minimum 0.10%
			   requiredProposalPercentStakeTimes1000 -= 100; // Decrease by 0.10%
			}

		emit RequiredProposalPercentStakeChanged(requiredProposalPercentStakeTimes1000);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L132-L146

```solidity
file: blob/main/src/dao/DAOConfig.sol

149   function changeMaxPendingTokensForWhitelisting(bool increase) external onlyOwner
		{
		if (increase)
			{
			if (maxPendingTokensForWhitelisting < 12)
				maxPendingTokensForWhitelisting += 1;
			}
		else
			{
			if (maxPendingTokensForWhitelisting > 3)
				maxPendingTokensForWhitelisting -= 1;
			}

		emit MaxPendingTokensForWhitelistingChanged(maxPendingTokensForWhitelisting);
		}


```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L149-L163

```solidity
file: blob/main/src/dao/DAOConfig.sol

166  function changeArbitrageProfitsPercentPOL(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (arbitrageProfitsPercentPOL < 45)
                arbitrageProfitsPercentPOL += 5;
            }
        else
            {
            if (arbitrageProfitsPercentPOL > 15)
                arbitrageProfitsPercentPOL -= 5;
            }

		emit ArbitrageProfitsPercentPOLChanged(arbitrageProfitsPercentPOL);
		}


```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L166-L180

```solidity 
file: blob/main/src/dao/DAOConfig.sol

183  function changeUpkeepRewardPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (upkeepRewardPercent < 10)
                upkeepRewardPercent += 1;
            }
        else
            {
            if (upkeepRewardPercent > 1)
                upkeepRewardPercent -= 1;
            }

		emit UpkeepRewardPercentChanged(upkeepRewardPercent);
        }

```


https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L183-L197

Issue Description - In the finalizeBallot() function, the state variable startExchangeApproved is read directly from storage on line 79 when emitting the event BallotFinalized. This is inefficient, as it incurs an extra storage read operation that is unnecessary.

Proposed Optimization - Cache the current value of startExchangeApproved in a local stack variable before emitting the event and updating storage. This avoids an unnecessary re-read of the storage.

```solidity
file: blob/main/src/launch/BootstrapBallot.sol

69   function finalizeBallot() external nonReentrant
		{
		require( ! ballotFinalized, "Ballot has already been finalized" );
		require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );

		if ( startExchangeYes > startExchangeNo )
			{
			exchangeConfig.initialDistribution().distributionApproved();
			exchangeConfig.dao().pools().startExchangeApproved();

			startExchangeApproved = true;
			}

		emit BallotFinalized(startExchangeApproved);

		ballotFinalized = true;
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L69-L85


cache state variable in function changeMaximumWhitelistedPools() and changeMaximumInternalSwapPercentTimes1000()

```solidity
file: blob/main/src/pools/PoolsConfig.sol

77   function changeMaximumWhitelistedPools(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maximumWhitelistedPools < 100)
                maximumWhitelistedPools += 10;
            }
        else
            {
            if (maximumWhitelistedPools > 20)
                maximumWhitelistedPools -= 10;
            }

		emit MaximumWhitelistedPoolsChanged(maximumWhitelistedPools);
        }

94   function changeMaximumInternalSwapPercentTimes1000(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maximumInternalSwapPercentTimes1000 < 2000)
                maximumInternalSwapPercentTimes1000 += 250;
            }
        else
            {
            if (maximumInternalSwapPercentTimes1000 > 250)
                maximumInternalSwapPercentTimes1000 -= 250;
            }

		emit maximumInternalSwapPercentTimes1000Changed(maximumInternalSwapPercentTimes1000);
        }

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L77-L91

cache this state variable priceFeedModificationCooldownExpiration in stack variable to save gas 

```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

47  function setPriceFeed( uint256 priceFeedNum, IPriceFeed newPriceFeed ) public onlyOwner
		{
		// If the required cooldown is not met, simply return without reverting so that the original proposal can be finalized and new setPriceFeed proposals can be made.
		if ( block.timestamp < priceFeedModificationCooldownExpiration )
			return;

		if ( priceFeedNum == 1 )
			priceFeed1 = newPriceFeed;
		else if ( priceFeedNum == 2 )
			priceFeed2 = newPriceFeed;
		else if ( priceFeedNum == 3 )
			priceFeed3 = newPriceFeed;

		priceFeedModificationCooldownExpiration = block.timestamp + priceFeedModificationCooldown;
		emit PriceFeedSet(priceFeedNum, newPriceFeed);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L47-L62

cache state variable in function changeMaximumPriceFeedPercentDifferenceTimes1000() and changePriceFeedModificationCooldown()

```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

65   function changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase) public onlyOwner
		{
        if (increase)
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 < 7000)
                maximumPriceFeedPercentDifferenceTimes1000 += 500;
            }
        else
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 > 1000)
                maximumPriceFeedPercentDifferenceTimes1000 -= 500;
            }

		emit MaximumPriceFeedPercentDifferenceChanged(maximumPriceFeedPercentDifferenceTimes1000);
		}

82   function changePriceFeedModificationCooldown(bool increase) public onlyOwner
		{
        if (increase)
            {
            if (priceFeedModificationCooldown < 45 days)
                priceFeedModificationCooldown += 5 days;
            }
        else
            {
            if (priceFeedModificationCooldown > 30 days)
                priceFeedModificationCooldown -= 5 days;
            }

		emit SetPriceFeedCooldownChanged(priceFeedModificationCooldown);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L65-L79

cache state variable in the following function 

```solidity
file:  blob/main/src/rewards/RewardsConfig.sol

36   
	function changeRewardsEmitterDailyPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (rewardsEmitterDailyPercentTimes1000 < 2500)
                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 + 250;
            }
        else
            {
            if (rewardsEmitterDailyPercentTimes1000 > 250)
                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 - 250;
            }

		emit RewardsEmitterDailyPercentChanged(rewardsEmitterDailyPercentTimes1000);
        }

	function changeEmissionsWeeklyPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (emissionsWeeklyPercentTimes1000 < 1000)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000 + 250;
            }
        else
            {
            if (emissionsWeeklyPercentTimes1000 > 250)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000 - 250;
            }

		emit EmissionsWeeklyPercentChanged(emissionsWeeklyPercentTimes1000);
        }


	function changeStakingRewardsPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (stakingRewardsPercent < 75)
                stakingRewardsPercent = stakingRewardsPercent + 5;
            }
        else
            {
            if (stakingRewardsPercent > 25)
                stakingRewardsPercent = stakingRewardsPercent - 5;
            }

		emit StakingRewardsPercentChanged(stakingRewardsPercent);
        }


	function changePercentRewardsSaltUSDS(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (percentRewardsSaltUSDS < 25)
                percentRewardsSaltUSDS = percentRewardsSaltUSDS + 5;
            }
        else
            {
            if (percentRewardsSaltUSDS > 5)
                percentRewardsSaltUSDS = percentRewardsSaltUSDS - 5;
            }

		emit PercentRewardsSaltUSDSChanged(percentRewardsSaltUSDS);
        }

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L36-L100

in function _possiblyBurnUSDS)() cache the state variable usdsThatShouldBeBurned

```solidity
file: blob/main/src/stable/Liquidizer.sol

101   function _possiblyBurnUSDS() internal
		{
		// Check if there is USDS to burn
		if ( usdsThatShouldBeBurned == 0 )
			return;

		uint256 usdsBalance = usds.balanceOf(address(this));
		if ( usdsBalance >= usdsThatShouldBeBurned )
			{
			// Burn only up to usdsThatShouldBeBurned.
			// Leftover USDS will be kept in this contract in case it needs to be burned later.
			_burnUSDS( usdsThatShouldBeBurned );
    		usdsThatShouldBeBurned = 0;
			}
		else
			{
			// The entire usdsBalance will be burned - but there will still be an outstanding balance to burn later
			_burnUSDS( usdsBalance );
			usdsThatShouldBeBurned -= usdsBalance;

			// As there is a shortfall in the amount of USDS that can be burned, liquidate some Protocol Owned Liquidity and
			// send the underlying tokens here to be swapped to USDS
			dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
			dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);
			}
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L101-L126

cache state variable in the following function 

```solidity
file:  blob/main/src/stable/StableConfig.sol

47   function changeRewardPercentForCallingLiquidation(bool increase) external onlyOwner
        {
        if (increase)
            {
			// Don't increase rewardPercentForCallingLiquidation if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

            if (remainingRatioAfterReward >= 105 && rewardPercentForCallingLiquidation < 10)
                rewardPercentForCallingLiquidation += 1;
            }
        else
            {
            if (rewardPercentForCallingLiquidation > 5)
                rewardPercentForCallingLiquidation -= 1;
            }

		emit RewardPercentForCallingLiquidationChanged(rewardPercentForCallingLiquidation);
        }


	function changeMaxRewardValueForCallingLiquidation(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maxRewardValueForCallingLiquidation < 1000 ether)
                maxRewardValueForCallingLiquidation += 100 ether;
            }
        else
            {
            if (maxRewardValueForCallingLiquidation > 100 ether)
                maxRewardValueForCallingLiquidation -= 100 ether;
            }

		emit MaxRewardValueForCallingLiquidationChanged(maxRewardValueForCallingLiquidation);
        }


	function changeMinimumCollateralValueForBorrowing(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minimumCollateralValueForBorrowing < 5000 ether)
                minimumCollateralValueForBorrowing += 500 ether;
            }
        else
            {
            if (minimumCollateralValueForBorrowing > 1000 ether)
                minimumCollateralValueForBorrowing -= 500 ether;
            }

		emit MinimumCollateralValueForBorrowingChanged(minimumCollateralValueForBorrowing);
        }


	function changeInitialCollateralRatioPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (initialCollateralRatioPercent < 300)
                initialCollateralRatioPercent += 25;
            }
        else
            {
            if (initialCollateralRatioPercent > 150)
                initialCollateralRatioPercent -= 25;
            }

		emit InitialCollateralRatioPercentChanged(initialCollateralRatioPercent);
        }


	function changeMinimumCollateralRatioPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minimumCollateralRatioPercent < 120)
                minimumCollateralRatioPercent += 1;
            }
        else
            {
			// Don't decrease the minimumCollateralRatioPercent if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation;

            if (remainingRatioAfterReward >= 105 && minimumCollateralRatioPercent > 110)
                minimumCollateralRatioPercent -= 1;
            }

		emit MinimumCollateralRatioPercentChanged(minimumCollateralRatioPercent);
        }


	function changePercentArbitrageProfitsForStablePOL(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (percentArbitrageProfitsForStablePOL < 10)
                percentArbitrageProfitsForStablePOL += 1;
            }
        else
            {
            if (percentArbitrageProfitsForStablePOL > 1)
                percentArbitrageProfitsForStablePOL -= 1;
            }

		emit PercentArbitrageProfitsForStablePOLChanged(percentArbitrageProfitsForStablePOL);
        }
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L47-L152 

cache state variabl in the following function 

```solidity
file: blob/main/src/staking/StakingConfig.sol

34   function changeMinUnstakeWeeks(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minUnstakeWeeks < 12)
                minUnstakeWeeks += 1;
            }
        else
            {
            if (minUnstakeWeeks > 1)
                minUnstakeWeeks -= 1;
            }

		emit MinUnstakeWeeksChanged(minUnstakeWeeks);
        }


	function changeMaxUnstakeWeeks(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maxUnstakeWeeks < 108)
                maxUnstakeWeeks += 8;
            }
        else
            {
            if (maxUnstakeWeeks > 20)
                maxUnstakeWeeks -= 8;
            }

		emit MaxUnstakeWeeksChanged(maxUnstakeWeeks);
        }


	function changeMinUnstakePercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minUnstakePercent < 50)
                minUnstakePercent += 5;
            }
        else
            {
            if (minUnstakePercent > 10)
                minUnstakePercent -= 5;
            }

		emit MinUnstakePercentChanged(minUnstakePercent);
        }


	function changeModificationCooldown(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (modificationCooldown < 6 hours)
                modificationCooldown += 15 minutes;
            }
        else
            {
            if (modificationCooldown > 15 minutes)
                modificationCooldown -= 15 minutes;
            }

		emit ModificationCooldownChanged(modificationCooldown);
        }
    }
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L34-L99


## [G-04] Use assembly to reuse memory space when making more than one external call

Issue Description - When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

Proposed Optimization - Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

Estimated Gas Savings - Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.

```solidity

See the example below;
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}
contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);
        uint256 res = res1 + res2;
        return res;
    }
}
contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }
            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)
            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)
            // add results
            let res := add(res1, res2)
            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}

```
We save approximately 2,000 gas by using the scratch space to store the function selector and its arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

Code Snippets:

```solidity
file:  blob/main/src/Upkeep.sol

132     uint256 amountA = pools.depositSwapWithdraw( weth, tokenA, wethAmountPerToken, 0, block.timestamp );
133		uint256 amountB = pools.depositSwapWithdraw( weth, tokenB, wethAmountPerToken, 0, block.timestamp );

136     tokenA.safeTransfer( address(dao), amountA );
137		tokenB.safeTransfer( address(dao), amountB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L132

```solidity 
file: blob/main/src/dao/DAO.sol

160    poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
161	   poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );

203   else if ( ballot.ballotType == BallotType.SET_CONTRACT )
      proposals.createConfirmationProposal( string.concat(ballot.ballotName, "_confirm"), BallotType.CONFIRM_SET_CONTRACT, ballot.address1, "", ballot.description );

	// Once an initial setWebsiteURL proposal passes, it automatically starts a second confirmation ballot (to prevent last minute approvals)
      else if ( ballot.ballotType == BallotType.SET_WEBSITE_URL )
    proposals.createConfirmationProposal( string.concat(ballot.ballotName, "_confirm"), BallotType.CONFIRM_SET_WEBSITE_URL, address(0), ballot.string1, ballot.description );


254  poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255  poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

375  tokenA.safeTransfer( address(liquidizer), reclaimedA );
376  tokenB.safeTransfer( address(liquidizer), reclaimedB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L160-L160


```solidity
file: blob/main/src/pools/Pools.sol

161   tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
162	  tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );

196   tokenA.safeTransfer( msg.sender, reclaimedA );
197	  tokenB.safeTransfer( msg.sender, reclaimedB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L161

```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

172   wbtc.safeTransfer( msg.sender, rewardedWBTC );
173	  weth.safeTransfer( msg.sender, rewardedWETH );

176   wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );
177	  weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L172-L173

```solidity
file: blob/main/src/stable/Liquidizer.sol

123   dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
124	  dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);

139   PoolUtils._placeInternalSwap(pools, wbtc, usds, wbtc.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );
140	  PoolUtils._placeInternalSwap(pools, weth, usds, weth.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );
141	  PoolUtils._placeInternalSwap(pools, dai, usds, dai.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );  

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L123

```solidity
file: blob/main/src/staking/Liquidity.sol

88  tokenA.safeTransferFrom(msg.sender, address(this), maxAmountA );
89	tokenB.safeTransferFrom(msg.sender, address(this), maxAmountB );

96  tokenA.approve( address(pools), maxAmountA );
97	tokenB.approve( address(pools), maxAmountB );

131 tokenA.safeTransfer( msg.sender, reclaimedA );
132	tokenB.safeTransfer( msg.sender, reclaimedB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L88-L89

## [G-05] Using delete statement can save gas

```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

184  usdsBorrowedByUsers[wallet] = 0;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L184

```solidity
file: blob/main/src/price_feed/CoreUniswapFeed.sol

54   secondsAgo[1] = 0; // to (now)

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54



## [G-06] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file: blob/main/src/Salt.sol

27   uint256 balance = balanceOf( address(this) );

28   burn( address(this), balance );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L27

```solidity
file: blob/main/src/Upkeep.sol

97   require(msg.sender == address(this), "Only callable from within the same contract");

147  uint256 wethBalance = weth.balanceOf( address(this) );

160  uint256 wethBalance = weth.balanceOf( address(this) );

173  uint256 wethBalance = weth.balanceOf( address(this) );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97

```solidity
file:  blob/main/src/dao/DAO.sol

170    if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )

246    uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );

300    uint256 depositedWETH = pools.depositedUserBalance(address(this), weth );

307    withdrawnAmount = weth.balanceOf( address(this) );

365    uint256 liquidityHeld = collateralAndLiquidity.userShareForPool( address(this), poolID );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L170

```solidity
file:  blob/main/src/pools/Pools.sol

161    tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );

162    tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );

212    token.safeTransferFrom(msg.sender, address(this), amount );

385    swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

397    swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L161

```solidity
file: blob/main/src/stable/Liquidizer.sol

107    uint256 usdsBalance = usds.balanceOf(address(this));

139    PoolUtils._placeInternalSwap(pools, wbtc, usds, wbtc.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );

140	   PoolUtils._placeInternalSwap(pools, weth, usds, weth.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );

141	   PoolUtils._placeInternalSwap(pools, dai, usds, dai.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L107


## [G-07] Emit local variables instead of state variable (Save ~100 Gas)

```solidity
file: blob/main/src/ManagedWallet.sol

54   emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);

83   emit WalletChange(mainWallet, confirmationWallet);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L54


```solidity
file: blob/main/src/stable/Liquidizer.sol

87   emit incrementedBurnableUSDS(usdsThatShouldBeBurned);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L87

```solidity
file: blob/main/src/AccessManager.sol

67    emit AccessGranted( msg.sender, geoVersion );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L67

```solidity
file: blob/main/src/dao/DAOConfig.sol

94  emit PercentPolRewardsBurnedChanged(percentPolRewardsBurned);

111  emit BaseBallotQuorumPercentChanged(baseBallotQuorumPercentTimes1000);

128  emit BallotDurationChanged(ballotMinimumDuration);

145  emit RequiredProposalPercentStakeChanged(requiredProposalPercentStakeTimes1000);

162  emit MaxPendingTokensForWhitelistingChanged(maxPendingTokensForWhitelisting);

179  emit ArbitrageProfitsPercentPOLChanged(arbitrageProfitsPercentPOL);

196  emit UpkeepRewardPercentChanged(upkeepRewardPercent);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L94


```solidity
file: blob/main/src/launch/BootstrapBallot.sol

82   emit BallotFinalized(startExchangeApproved);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L82

```solidity
file: blob/main/src/pools/PoolsConfig.sol

90  emit MaximumWhitelistedPoolsChanged(maximumWhitelistedPools);

107  emit maximumInternalSwapPercentTimes1000Changed(maximumInternalSwapPercentTimes1000);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L90


## [G-08] Avoid calling the same internal function multiple times

Calling an internal function will cost at least ~16-20 gas (JUMP to internal function instructions and JUMP back to original instructions). If you call the same internal function multiple times you can save gas by caching the return value of the internal function.

```solidity
file: blob/main/src/dao/DAO.sol

119  if ( winningVote == Vote.INCREASE )
			_executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
		else if ( winningVote == Vote.DECREASE )
			_executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L119-L122

```solidity


    address sender = _executeParameterChange();

	sender( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
		else if ( winningVote == Vote.DECREASE )
			sender( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );

```

## [G-09] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

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
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

Code Snippets:

```solidity
file: blob/main/src/AccessManager.sol

43   require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

63   require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43

```solidity
file: blob/main/src/ExchangeConfig.sol

51   require( address(dao) == address(0), "setContracts can only be called once" );

64   require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" )

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51

```solidity
file: blob/main/src/ManagedWallet.sol

44    require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

45    require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );

46    require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );

49    require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );

61    require( msg.sender == confirmationWallet, "Invalid sender" );

76    require( msg.sender == proposedMainWallet, "Invalid sender" );

77    require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44

```solidity
file: blob/main/src/SigningTools.sol

13   require( signature.length == 65, "Invalid signature length" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L13


```solidity
file: blob/main/src/Upkeep.sol

97    require(msg.sender == address(this), "Only callable from within the same contract");

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97

```solidity
file: blob/main/src/dao/DAO.sol

247   require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

251   require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

297   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );

318   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );

329   require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );

362   require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247

```solidity
file: blob/main/src/dao/Proposals.sol

83   require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

92   require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

95   require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );

98   require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );

102  require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );

103  require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );

124  require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

132  require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

164  require( address(token) != address(0), "token cannot be address(0)" );

165  require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); 

167  require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );

168   require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );

169   require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );

182   require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );

183   require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );

184   require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );

185   require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );

186	  require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );

187	  require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

196   require( wallet != address(0), "Cannot send SALT to address(0)" );

203   require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

215   require( contractAddress != address(0), "Contract address cannot be address(0)" );

224   require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

233   require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

242   require( newAddress != address(0), "Proposed address cannot be address(0)" );

251   require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );

264   require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

268   require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );

270   require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );

277   require( userVotingPower > 0, "Staked SALT required to vote" );

321   require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L83


```solidity
file: blob/main/src/launch/Airdrop.sol

48   require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

49   require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );

58   require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );

59   require( ! claimingAllowed, "Claiming is already allowed" );

60   require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

76   require( claimingAllowed, "Claiming is not allowed yet" );

77   require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );

78   require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48

```solidity
file: blob/main/src/launch/BootstrapBallot.sol

36   require( ballotDuration > 0, "ballotDuration cannot be zero" );

50   require( ! hasVoted[msg.sender], "User already voted" );

54   require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );

71   require( ! ballotFinalized, "Ballot has already been finalized" );

72   require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L36

```solidity
file: blob/main/src/launch/InitialDistribution.sol

52   require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );

53   require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52

```solidity
file:  blob/main/src/pools/Pools.sol

72    require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );

83    require(block.timestamp <= deadline, "TX EXPIRED");

142   require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );

143   require( exchangeIsLive, "The exchange is not yet live" );

144   require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );

146   require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );

147   require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );

158   require( addedLiquidity >= minLiquidityReceived, "Too little liquidity received" );

172   require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

173   require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

187   require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

193   require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

207   require( amount > PoolUtils.DUST, "Deposit amount too small");

221   require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

222   require( amount > PoolUtils.DUST, "Withdraw amount too small");

243   require((reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves before swap");

271   require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap");

353   require( swapAmountOut >= minAmountOut, "Insufficient resulting token amount" );

371   require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72


```solidity
file: blob/main/src/pools/PoolsConfig.sol

47   require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );

48   require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");

136  require(address(pair.tokenA) != address(0) && address(pair.tokenB) != address(0), "This poolID does not exist");

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L47


```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

39    require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );

183   require (price != 0, "Invalid BTC price" );

195   require (price != 0, "Invalid ETH price" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L39

```solidity
file: blob/main/src/rewards/RewardsEmitter.sol

63    require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );

84    require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L63

```solidity
file: blob/main/src/rewards/SaltRewards.sol

60    require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

108   require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );

119   require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );

120   require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L60


```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

83   require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

84   require( collateralToWithdraw <= maxWithdrawableCollateral(msg.sender), "Excessive collateralToWithdraw" );
 
97   require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

98   require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

99	 require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );

117  require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );

118	 require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );

119	 require( amountRepaid > 0, "Cannot repay zero amount" );

142  require( wallet != msg.sender, "Cannot liquidate self" );

145  require( canUserBeLiquidated(wallet), "User cannot be liquidated" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L83

```solidity
file: blob/main/src/staking/Liquidity.sol

42   require(block.timestamp <= deadline, "TX EXPIRED");

85   require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

124  require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

148  require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be deposited via Liquidity.depositLiquidityAndIncreaseShare" );

159  require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be withdrawn via Liquidity.withdrawLiquidityAndClaim" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L42


```solidity
file: blob/main/src/staking/Staking.sol

43   require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );

62   require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" );

88   require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" );

89	 require( block.timestamp < u.completionTime, "Unstakes that have already completed cannot be cancelled" );

90	 require( msg.sender == u.wallet, "Sender is not the original staker" );

104  require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" );

105	 require( block.timestamp >= u.completionTime, "Unstake has not completed yet" );

106	 require( msg.sender == u.wallet, "Sender is not the original staker" );

132  require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );

153  require(end >= start, "Invalid range: end cannot be less than start");

157  require(userUnstakes.length > end, "Invalid range: end is out of bounds");

158  require(start < userUnstakes.length, "Invalid range: start is out of bounds");

204  require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );

205  require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L43


```solidity
file: blob/main/src/staking/StakingRewards.sol

59    require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );

60    require( increaseShareAmount != 0, "Cannot increase zero share" );

67    require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

99    require( decreaseShareAmount != 0, "Cannot decrease zero share" );

102   require( decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share" );

107   require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

190   require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L59


## [G-10] Shorten arrays with inline assembly

Issue Description - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

Proposed Optimization - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

Estimated Gas Savings - Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.


```solidity

function shorten(uint[] storage array, uint newLen) internal {
  assembly {
    sstore(array_slot, newLen)
  }
}
// Rather than:
function shorten(uint[] storage array, uint newLen) internal {
  uint[] memory newArray = new uint[](newLen);
  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }
  delete array;
  array = newArray;
}

```
Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

Code Snippet:

```solidity
file: blob/main/src/dao/DAO.sol

261   AddedReward[] memory addedRewards = new AddedReward[](2);

332   bytes32[] memory poolIDs = new bytes32[](2);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L261


```solidity
file: blob/main/src/price_feed/CoreUniswapFeed.sol

52   uint32[] memory secondsAgo = new uint32[](2);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L52

```solidity
file: blob/main/src/rewards/RewardsEmitter.sol

109   AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

144   uint256[] memory rewards = new uint256[]( pools.length );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L109

```solidity
file:  blob/main/src/rewards/SaltRewards.sol

49   AddedReward[] memory addedRewards = new AddedReward[](1);

63   AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

86   AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );

98   AddedReward[] memory addedRewards = new AddedReward[](1);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L49

```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

313  address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);

337  address[] memory resizedLiquidatableUsers = new address[](count);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L313

```solidity
file:  blob/main/src/staking/Staking.sol

160    Unstake[] memory unstakes = new Unstake[](end - start + 1);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L160


```solidity
file: blob/main/src/AccessManager.sol

53   bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L53

```solidity
file:  blob/main/src/dao/Proposals.sol

251   require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251

```solidity
file: blob/main/src/launch/BootstrapBallot.sol

53   bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L53


```solidity
file: blob/main/src/pools/PoolUtils.sol

25   return keccak256(abi.encodePacked(address(tokenB), address(tokenA)));

27   return keccak256(abi.encodePacked(address(tokenA), address(tokenB)));'

36   return (keccak256(abi.encodePacked(address(tokenB), address(tokenA))), true);

38   return (keccak256(abi.encodePacked(address(tokenA), address(tokenB))), false);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L25

## [G-11] Don’t make variables public unless necessary

Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be public.

Code Snippets:

```solidity
file: blob/main/src/ExchangeConfig.sol

24   IDAO public dao; // can only be set once

25   IUpkeep public upkeep; // can only be set once

26   IInitialDistribution public initialDistribution; // can only be set once

27   IAirdrop public airdrop; // can only be set once

30   VestingWallet public teamVestingWallet;	

31   VestingWallet public daoVestingWallet;

33   IAccessManager public accessManager;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L24


```solidity
file: blob/main/src/ManagedWallet.sol

20   address public mainWallet;

21   address public confirmationWallet;

24   address public proposedMainWallet;

25   address public proposedConfirmationWallet;

28   uint256 public activeTimelock;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L20


```solidity
file: blob/main/src/stable/StableConfig.sol

20   uint256 public rewardPercentForCallingLiquidation = 5;

24   uint256 public maxRewardValueForCallingLiquidation = 500 ether;

29   uint256 public minimumCollateralValueForBorrowing = 2500 ether;

34   uint256 public initialCollateralRatioPercent = 200;

39   uint256 public minimumCollateralRatioPercent = 110;

44   uint256 public percentArbitrageProfitsForStablePOL = 5;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L20

```solidity 
file: blob/main/src/Upkeep.sol

63  uint256 public lastUpkeepTimeEmissions;

64	uint256 public lastUpkeepTimeRewardsEmitters;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L63

```solidity
file: blob/main/src/dao/DAOConfig.sol

24   uint256 public bootstrappingRewards = 200000 ether;

28   uint256 public percentPolRewardsBurned = 50;

40   uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000;

45   uint256 public ballotMinimumDuration = 10 days;

49   uint256 public requiredProposalPercentStakeTimes1000 = 500; 

53   uint256 public maxPendingTokensForWhitelisting = 5;

57   uint256 public arbitrageProfitsPercentPOL = 20;

61   uint256 public upkeepRewardPercent = 5;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L24

```solidity
file: blob/main/src/launch/Airdrop.sol

26   bool public claimingAllowed;

29   mapping(address=>bool) public claimed;

32   uint256 public saltAmountForEachUser;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L26

```solidity
file: blob/main/src/launch/BootstrapBallot.sol

21   bool public ballotFinalized;

22   bool public startExchangeApproved;

25   mapping(address=>bool) public hasVoted;

29   uint256 public startExchangeYes;

30   uint256 public startExchangeNo;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L21


```solidity
file: blob/main/src/pools/PoolsConfig.sol

32   mapping(bytes32=>TokenPair) public underlyingPoolTokens;

37   uint256 public maximumWhitelistedPools = 50;

41   uint256 public maximumInternalSwapPercentTimes1000 = 1000;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L32


```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

19   IPriceFeed public priceFeed1; 

20   IPriceFeed public priceFeed2; 

21   IPriceFeed public priceFeed3;

24   uint256 public priceFeedModificationCooldownExpiration;

29   uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; 

34   uint256 public priceFeedModificationCooldown = 35 days;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L19

```solidity
file: blob/main/src/rewards/RewardsConfig.sol   

19   uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;

23   uint256 public emissionsWeeklyPercentTimes1000 = 500; 

27   uint256 public stakingRewardsPercent = 50;

33   uint256 public percentRewardsSaltUSDS = 10;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L19

```solidity
file: blob/main/src/AccessManager.sol
 
29   uint256 public geoVersion;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L29

##  [G-12] When using storage instead of memory, we should cache any fields that need to be re-read in stack variables

```solidity
file: blob/main/src/dao/Proposals.sol

134   Ballot storage ballot = ballots[ballotID];

		// Remove finalized whitelist token ballots from the list of open whitelisting proposals
		if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
			_openBallotsForTokenWhitelisting.remove( ballotID );

		// Remove from the list of all open ballots
		_allOpenBallots.remove( ballotID );

		ballot.ballotIsLive = false;

		// Indicate that the user who posted the proposal no longer has an active proposal
		address userThatPostedBallot = _usersThatProposedBallots[ballotID];
		_userHasActiveProposal[userThatPostedBallot] = false;

		delete openBallotsByName[ballot.ballotName];

		emit BallotFinalized(ballotID);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L134-L152


```solidity
file:  blob/main/src/dao/Proposals.sol

344  		mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];

		Ballot memory ballot = ballots[ballotID];
		if ( ballot.ballotType == BallotType.PARAMETER )
			return votes[Vote.INCREASE] + votes[Vote.DECREASE] + votes[Vote.NO_CHANGE];
		else
			return votes[Vote.YES] + votes[Vote.NO];
		}


```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L344-L351  

```solidity
file: blob/main/src/dao/Proposals.sol

366  	mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];

		uint256 increaseTotal = votes[Vote.INCREASE];
		uint256 decreaseTotal = votes[Vote.DECREASE];
		uint256 noChangeTotal = votes[Vote.NO_CHANGE];
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L366-L370


```solidity
file:  blob/main/src/pools/Pools.sol

92  		PoolReserves storage reserves = _poolReserves[poolID];
		uint256 reserve0 = reserves.reserve0;
		uint256 reserve1 = reserves.reserve1;

		// If either reserve is zero then consider the pool to be empty and that the added liquidity will become the initial token ratio
		if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
			{
			// Update the reserves
			reserves.reserve0 += uint128(maxAmount0);
			reserves.reserve1 += uint128(maxAmount1);

			// Default liquidity will be the addition of both maxAmounts in case one of them is much smaller (has smaller decimals)
			return ( maxAmount0, maxAmount1, (maxAmount0 + maxAmount1) );
			}

		// Add liquidity to the pool proportional to the current existing token reserves in the pool.
		// First, try the proportional amount of tokenB for the given maxAmountA
		uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;

		// proportionalB too large for the specified maxAmountB?
		if ( proportionalB > maxAmount1 )
			{
			// Use maxAmountB and a proportional amount for tokenA instead
			addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
			addedAmount1 = maxAmount1;
			}
		else
			{
			addedAmount0 = maxAmount0;
			addedAmount1 = proportionalB;
			}

		// Update the reserves
		reserves.reserve0 += uint128(addedAmount0);
		reserves.reserve1 += uint128(addedAmount1);

		// Determine the amount of liquidity that will be given to the user to reflect their share of the total collateralAndLiquidity.
		// Use whichever added amount was larger to maintain better numeric resolution.
		// Rounded down in favor of the protocol.
		if ( addedAmount0 > addedAmount1)
			addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
		else
			addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
		}
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L92-L135


```solidity
file:  blob/main/src/pools/Pools.sol

178    PoolReserves storage reserves = _poolReserves[poolID];
		reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
		reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;

		reserves.reserve0 -= uint128(reclaimedA);
		reserves.reserve1 -= uint128(reclaimedB);

		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L178-L187

```solidity
file:  blob/main/src/pools/Pools.sol

239        PoolReserves storage reserves = _poolReserves[poolID];
        uint256 reserve0 = reserves.reserve0;
        uint256 reserve1 = reserves.reserve1;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L239-L241

```solidity
file:  blob/main/src/staking/StakingRewards.sol

62  	UserShareInfo storage user = _userShareInfo[wallet][poolID];

		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

			// Update the cooldown expiration for future transactions
			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
			}

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L62-L71

## [G-13] Use constants for variables that don’t change (Save a storage SLOT: 2200 Gas)

```solidity
file: blob/main/src/dao/DAOConfig.sol

45   uint256 public ballotMinimumDuration = 10 days;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L45

The variable ballotMinimumDuration should be declared as a constant variable since it does not change, from the naming, it would seem the intention was to actually make it a constant.

```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

34   uint256 public priceFeedModificationCooldown = 35 days;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L34

## [G-14] Validate all parameters first before performing any other operations if there’s no side effects

```solidity
file  blob/main/src/launch/BootstrapBallot.sol
 
56   	if ( voteStartExchangeYes )
			startExchangeYes++;
		else
			startExchangeNo++;

		hasVoted[msg.sender] = true;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#l56-L61

```solidity
file: blob/main/src/pools/Pools.sol

112  if ( proportionalB > maxAmount1 )
			{
			// Use maxAmountB and a proportional amount for tokenA instead
			addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
			addedAmount1 = maxAmount1;
			}
		else
			{
			addedAmount0 = maxAmount0;
			addedAmount1 = proportionalB;
			}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L112-L122


```solidity
file:  blob/main/src/rewards/Emissions.sol
 
44  	if ( timeSinceLastUpkeep == 0 )
			return;

		// Cap the timeSinceLastUpkeep at one week (if for some reason it has been longer).
		// This will cap the emitted rewards at a default of 0.50% in this transaction.
		if ( timeSinceLastUpkeep >= MAX_TIME_SINCE_LAST_UPKEEP )
	
```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L44-L49


```solidity
file: blob/main/src/staking/Liquidity.sol

92   if ( useZapping )
			(maxAmountA, maxAmountB) = _dualZapInLiquidity(tokenA, tokenB, maxAmountA, maxAmountB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L92-L93


```solidity
file:  blob/main/src/staking/Liquidity.sol

110   if ( addedAmountA < maxAmountA )
			tokenA.safeTransfer( msg.sender, maxAmountA - addedAmountA );

		if ( addedAmountB < maxAmountB )
			tokenB.safeTransfer( msg.sender, maxAmountB - addedAmountB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L110-L114


```solidity
file:  blob/main/src/staking/StakingRewards.sol

64   if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L64-L65

## [G-15] Use the already cached variable instead of reading from storage (Save 1 SLOAD: 100 Gas)

use existingTotalShares memory variable instead of using totalShares[poolID]

```solidity
file: blob/main/src/staking/StakingRewards.sol
 
73  uint256 existingTotalShares = totalShares[poolID];

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L73

use the cached variable reserve0 instead of using the reserves.reserve0; 

and also use reserve1 instead of using  reserves.reserve1

```solidity 
file:  blob/main/src/pools/Pools.sol

93  		uint256 reserve0 = reserves.reserve0;
		uint256 reserve1 = reserves.reserve1;

		// If either reserve is zero then consider the pool to be empty and that the added liquidity will become the initial token ratio
		if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
			{
			// Update the reserves
			reserves.reserve0 += uint128(maxAmount0);
			reserves.reserve1 += uint128(maxAmount1);

			// Default liquidity will be the addition of both maxAmounts in case one of them is much smaller (has smaller decimals)
			return ( maxAmount0, maxAmount1, (maxAmount0 + maxAmount1) );
			}

		// Add liquidity to the pool proportional to the current existing token reserves in the pool.
		// First, try the proportional amount of tokenB for the given maxAmountA
		uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;

		// proportionalB too large for the specified maxAmountB?
		if ( proportionalB > maxAmount1 )
			{
			// Use maxAmountB and a proportional amount for tokenA instead
			addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
			addedAmount1 = maxAmount1;
			}
		else
			{
			addedAmount0 = maxAmount0;
			addedAmount1 = proportionalB;
			}

		// Update the reserves
		reserves.reserve0 += uint128(addedAmount0);
		reserves.reserve1 += uint128(addedAmount1);

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L93-L126


## [G-16] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

missed by bot

```solidity
file:  blob/main/src/AccessManager.sol

26     mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L26


```solidity
file: blob/main/src/dao/DAO.sol

67   mapping(string=>bool) public excludedCountries;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L67

```solidity
file: blob/main/src/dao/Proposals.sol

59   mapping(address=>bool) private _userHasActiveProposal;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L59

```solidity
file: blob/main/src/launch/Airdrop.sol

26  bool public claimingAllowed;

29  mapping(address=>bool) public claimed;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L26

```solidity
file: blob/main/src/launch/BootstrapBallot.sol

21   bool public ballotFinalized;

22   bool public startExchangeApproved;

25   mapping(address=>bool) public hasVoted;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L21


```solidity
file:  blob/main/src/pools/Pools.sol

43    bool public exchangeIsLive;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L43


```solidity
file: blob/main/src/price_feed/CoreUniswapFeed.sol

28   bool immutable public wbtc_wethFlipped;

29   bool immutable public weth_usdcFlipped;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L28

```solidity
file: blob/main/src/rewards/RewardsEmitter.sol

29  bool immutable isForCollateralAndLiquidity;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L29

## [G-17] Add unchecked {} for subtractions where the operands can’t underflow because of previous checks

Issue Description - The subtraction operation dust = normalizedTransferAmount - amount; does not have an unchecked block, even though underflow is not possible due to the previous check normalizedTransferAmount > amount. Adding an unchecked block helps avoid unnecessary checks.

Proposed Optimization - Add an unchecked block around the subtraction:

```solidity

unchecked {
  dust = normalizedTransferAmount - amount;
}

```
Estimated Gas Savings - Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction. With many such subtractions occurring across interactions, the total gas savings could be significant.

Code Snippets:

```solidity
file:  blob/main/src/Upkeep.sol

186    uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions;

207    uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeRewardsEmitters;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L186

```solidity
file: blob/main/src/arbitrage/ArbitrageSearch.sol

72    int256 profitMidpoint = int256(amountOut) - int256(midpoint);

86    int256 profitRightOfMidpoint = int256(amountOut) - int256(midpoint);

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L72

```solidity
file: blob/main/src/staking/StakingRewards.sol

133  claimableRewards = rewardsForAmount - virtualRewardsToRemove;

251  return rewardsShare - user.virtualRewards;

303  cooldowns[i] = cooldownExpiration - block.timestamp;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L133

```solidity
file: blob/main/src/dao/DAO.sol

345   uint256 remainingSALT = claimedSALT - amountToSendToTeam;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L345


```solidity
file: blob/main/src/pools/Pools.sol

290   arbitrageProfit = amountOut - arbitrageAmountIn;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L290

```solidity
file: blob/main/src/price_feed/CoreChainlinkFeed.sol

43    uint256 answerDelay = block.timestamp - _answerTimestamp;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L43

```solidity
file: blob/main/src/price_feed/PriceAggregator.sol

102   return x - y;

104   return y - x;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L102

```solidity
file: blob/main/src/rewards/RewardsConfig.sol

46    rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 - 250;

62    emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000 - 250;

79    stakingRewardsPercent = stakingRewardsPercent - 5;

96    percentRewardsSaltUSDS = percentRewardsSaltUSDS - 5;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L46

```solidity
file: blob/main/src/rewards/SaltRewards.sol

140   uint256 remainingRewards = saltRewardsToDistribute - directRewardsForSaltUSDS;

144   uint256 liquidityRewardsAmount = remainingRewards - stakingRewardsAmount;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L140

```solidity
file: blob/main/src/stable/CollateralAndLiquidity.sol

176  wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );

177  weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L176


```solidity
file: blob/main/src/stable/StableConfig.sol

52    uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

128   uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation;

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L52

```solidity
file: blob/main/src/staking/Liquidity.sol

111   tokenA.safeTransfer( msg.sender, maxAmountA - addedAmountA );

114   tokenB.safeTransfer( msg.sender, maxAmountB - addedAmountB );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L111

```solidity
file:

112   uint256 expeditedUnstakeFee = u.unstakedXSALT - u.claimableSALT;

179   return unstakesForUser( user, 0, unstakeIDs.length - 1 );

207   uint256 percentAboveMinimum = 100 - minUnstakePercent;

208   uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L112

```solidity
file: blob/main/src/pools/PoolMath.sol

162   shift = maximumMSB - 80;

192   uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );

203   swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L162

## [G-18] Avoid contract existence checks by using low level calls

Issue Description - Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls, even if the call had a return value. This wasted gas by performing a check that wasn’t necessary.

Proposed Optimization - For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:
contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}
//After:
contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```
Estimated Gas Savings - Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas total.

Code Snippets:

```solidity
file: blob/main/src/Upkeep.sol

132   uint256 amountA = pools.depositSwapWithdraw( weth, tokenA, wethAmountPerToken, 0, block.timestamp );

133   uint256 amountB = pools.depositSwapWithdraw( weth, tokenB, wethAmountPerToken, 0, block.timestamp );

152   uint256 amountOfWETH = wethBalance * stableConfig.percentArbitrageProfitsForStablePOL() / 100;
		_formPOL(usds, dai, amountOfWETH);

178   uint256 amountSALT = pools.depositSwapWithdraw( weth, salt, wethBalance, 0, block.timestamp );
		salt.safeTransfer(address(saltRewards), amountSALT);

233  uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L132

```solidity
file: blob/main/src/dao/DAO.sol

365   uint256 liquidityHeld = collateralAndLiquidity.userShareForPool( address(this), poolID );

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L365
 
```solidity
file: blob/main/src/dao/Proposals.sol
 
157   string memory ballotName = string.concat("parameter:", Strings.toString(parameterType) );'

```
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L157


