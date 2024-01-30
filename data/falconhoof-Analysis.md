# LOW

## L-01 typo in Pools::removeLiquidity()

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L170-L200

There is a typo in `removeLiquidity()` which allows the `reserve1` token (tokenB) to drop below the `DUST` value which can have negative consequences for the stability of the protocol and spot prices in Pools.
The highlighted require statement checks whether reserve0 is below DUST twice without checking reserve 1 when it should check both.


```
	function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
		{
        // SOME CODE

>>>     require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

        // SOME CODE
		}
```

### POC
Add test to Pools.t.sol and run:
```
	function test_Reserve1BelowDUST() public
    	{
		vm.startPrank(address(collateralAndLiquidity));
		IERC20 tokenA = new TestERC20("newToken", 18);
        IERC20 tokenB = new TestERC20("newToken", 18);
		vm.stopPrank();

		vm.prank(address(dao));
		poolsConfig.whitelistPool( pools,   tokenA, tokenB);

		uint256 transferAmountA = 100_000 ether;
		uint256 transferAmountB = 100 ether;

		vm.startPrank(address(collateralAndLiquidity));
    	tokenA.transfer(alice, transferAmountA);
    	tokenB.transfer(alice, transferAmountB);
		vm.stopPrank();

		// assert funds transferred correctly
		assertEq(transferAmountA, tokenA.balanceOf(alice));
		assertEq(transferAmountB, tokenB.balanceOf(alice));

		vm.startPrank(alice); // depositing tokens are 
		tokenA.approve(address(collateralAndLiquidity),type(uint256).max);
		tokenB.approve(address(collateralAndLiquidity),type(uint256).max);
		vm.stopPrank();

		uint256 aliceDeposit_TokenA = 100_000 ether;
		uint256 aliceDeposit_TokenB = 10 ether;

		// alice is the initial depositor
        vm.prank(alice);
		collateralAndLiquidity.depositLiquidityAndIncreaseShare( tokenA, tokenB, aliceDeposit_TokenA, aliceDeposit_TokenB, 0, block.timestamp, false );
		uint256 aliceLiquidity = collateralAndLiquidity.userShareForPool(alice, PoolUtils._poolID(tokenA, tokenB));

		vm.warp( block.timestamp + 1 hours);

		vm.prank(alice);
		collateralAndLiquidity.withdrawLiquidityAndClaim( tokenA, tokenB, aliceLiquidity - 100, 0, 0, block.timestamp );
		aliceLiquidity = collateralAndLiquidity.userShareForPool(alice, PoolUtils._poolID(tokenA, tokenB));
		(uint256 poolA, uint256 poolB) = pools.getPoolReserves(tokenA, tokenB);		
		assert(poolA == 100);
		assert(poolB < 100);
        }
```

### Recommendation
Update the typo as:
```diff
- require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), - "Insufficient reserves after liquidity removal");


+ require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

## L-02 no end time on proposals

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81-L118

There is no end time on created proposals which means that proposals don't expire. This runs a risk of a proposal lying dormant until some time in the future where a user executes the proposal and because of the changed conditions of the protocol at the future point in time, it could have extremely negative, unforeseen consequences.
An extremely negative outcome could arise from a non-malicious proposal which was simply forgotten about or a malicious user in the early life of the protocol could create a proposal to send salt to themsleves. The proposal does not reach quorum and sits dormant until such time in the future as `xSALT` requirements decrease so that the malicious has increased voting power to get the proposal over the line.

### Recommendation
Add a field to the ballot struct with a maxEndTime, after which the proposal cannot be executed. Populate the field when the ballot is created in `Proposals::_possiblyCreateProposal`:

```diff
struct Ballot { 
	uint256 ballotID;
	bool ballotIsLive;
	BallotType ballotType;
	string ballotName;
	address address1;
	uint256 number1;
	string string1;
	string description;
	uint256 ballotMinimumEndTime;
+   uint256 ballotMaximumEndTime;
    }
```

## L-03 no veto period on proposals

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L155-L291

There is no veto period on proposals, which is dangerous and can allow malicious propsoals to pass a vote and get implemented.
After a vote passes with a Quorum, there should be a period of review during which certain trusted users can veto the execution of the proposal. 
This is to protect lay users from voting through proposals which may be detrimental to them and the protocol at large by having more knowledgable users form a line of defense. 

### Recommendation
Add logic to the proposals and voting functionality to enact a veto period after a successful passing of a proposal where malicious proposals can be prevented from going through.

## L-04 returned chainlink prices should be checked for staleness

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L28-L60

There are not enough checks for price staleness in the `CoreChainlinkFeed::latestChainlinkPrice()`. The resulting price information may be outdated and affect accurate token pricing.

### Recommendation
Implement stale price checks in `latestChainlinkPrice()`

```diff
	function latestChainlinkPrice(AggregatorV3Interface chainlinkFeed) public view returns (uint256)
		{
		int256 price = 0;

		try chainlinkFeed.latestRoundData()
		returns (
-			uint80, // _roundID
+           uint80 _roundID,
			int256 _price,
			uint256, // _startedAt
			uint256 _answerTimestamp,
-			uint80 // _answeredInRound
+           uint80 _answeredInRound
		)
			{
+           require(rawPrice > 0, "Chainlink price <= 0");
+           require(updateTime != 0, "Incomplete round");
+           require(answeredInRound >= roundId, "Stale price");

			// Make sure that the Chainlink price update has occurred within its 60 minute heartbeat
			// https://docs.chain.link/data-feeds#check-the-timestamp-of-the-latest-answer
			uint256 answerDelay = block.timestamp - _answerTimestamp;

			if ( answerDelay <= MAX_ANSWER_DELAY )
				price = _price;
			else
				price = 0;
			}
		catch (bytes memory) // Catching any failure
			{
			// In case of failure, price will remain 0
			}

		if ( price < 0 )
			return 0;

		// Convert the 8 decimals from the Chainlink price to 18 decimals
		return uint256(price) * 10**10;
		}
```

## L-05 DAO should be able to open more than one proposal at a time

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81-L118

The DAO address creates confirmation proposals as part of the process for updating the `priceFeed` address, the `accessManager` addresses & the `websiteURL`. 
There is a check in `Proposals::_possiblyCreateProposal()` which prevents the DAO from creating more than one confirmation proposal at a time.
This is risky as it reduces the adaptability of the protocol; whereby if something needs updating quickly but there is already a live confirmation proposal there will be a delay in putting it to a vote.
Given that the DAO is trusted; this restriction should not apply to the DAO.

```
	function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
		{
        // Some Code

		_userHasActiveProposal[msg.sender] = true;

		// Some Code
		}

```

### Recommendation
Increase the amount of concurrent proposals which the DAO is allowed to have open.


## L-06 If user has 0 voting power, their previous vote should be decreased

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L259-L293

When `Proposals::castVote()` is called by a user, there is a check whether user has any voting power. If they do not then the function is reverted.


```
	function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{

        // SOME CODE

>>>		require( userVotingPower > 0, "Staked SALT required to vote" );

		// Remove any previous votes made by the user on the ballot
		UserVote memory lastVote = _lastUserVoteForBallot[ballotID][msg.sender];

		// Undo last vote and voting power
		if ( lastVote.votingPower > 0 )
			_votesCastForBallot[ballotID][lastVote.vote] -= lastVote.votingPower;

		// Update user's vote and voting power
		_votesCastForBallot[ballotID][vote] += userVotingPower;

		// Remember how the user voted in case they change their vote later
		_lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower );

		emit VoteCast(msg.sender, ballotID, vote, userVotingPower);
		}
```

### Recommendation
If user does not have any voting power, it should still be checked whether they previously voted on the ballot. If `YES`, their vote should be reduced to 0.

## L-07 If bootstrap ballot does not pass; it can be very costly to the protocol

Link: 
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L69-L85
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ExchangeConfig.sol#L48-L59

During the Launch phase, if the vote checked in `BootstrapBallot::finalizeBallot()` fails; it will be very costly to the protocol.
Firstly it will require the re-deployment of the contracts involved in the launch, i.e. `airdrop`, `initialDistribution`, `bootstrapBallot` & `SALT`.

```
	function finalizeBallot() external nonReentrant
		{
		require( ! ballotFinalized, "Ballot has already been finalized" );
		require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );

>>>		if ( startExchangeYes > startExchangeNo )
			{
			exchangeConfig.initialDistribution().distributionApproved();
			exchangeConfig.dao().pools().startExchangeApproved();

			startExchangeApproved = true;
			}

		emit BallotFinalized(startExchangeApproved);

		ballotFinalized = true;
		}
```

Secondly, because the launch contracts are set in `ExchangeConfig` via a function that can only be called once, `setContracts()` that contract will also need to be redeployed; which also means all the other contracts storing a reference to the `ExchangeConfig` contract have to be updated.

```
	function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
		{
		// setContracts is only called once (on deployment)
		require( address(dao) == address(0), "setContracts can only be called once" );

		dao = _dao;
		upkeep = _upkeep;
		initialDistribution = _initialDistribution;
		airdrop = _airdrop;
		teamVestingWallet = _teamVestingWallet;
		daoVestingWallet = _daoVestingWallet;
		}
```

### Recommendation
Add logic for a workaround whereby all these contracts needn't be re-deployed such as making the `ExchangeConfig::setContracts()` function callable by a trusted contract in the case the ballot fails.

# Non-Critical

## N-01 Redundant calculation of "sum"

Link: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L82-L137

In RewardsEmitter::performUpkeep a sum variable is set and updated but never used.

```
	function performUpkeep( uint256 timeSinceLastUpkeep ) external
		{

// SOME CODE

>>>		uint256 sum = 0;
		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];

			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;

			// Reduce the pending rewards so they are not sent again
			if ( amountToAddForPool != 0 )
				{
				pendingRewards[poolID] -= amountToAddForPool;

>>>				sum += amountToAddForPool;
				}

			// Specify the rewards that will be added for the specific pool
			addedRewards[i] = AddedReward( poolID, amountToAddForPool );
			}

		// Add the rewards so that they can later be claimed by the users proportional to their share of the StakingRewards derived contract.
		stakingRewards.addSALTRewards( addedRewards );
		}
```



### Time spent:
50 hours