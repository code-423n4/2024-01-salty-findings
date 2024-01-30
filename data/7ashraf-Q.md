
# Quality Assurance Report

## Summary


The QA report highlights several issues in the codebase. One critical issue ([L-01]) involves the addition of a cooldown period even when the price is not changed, leading to false emissions. Additionally, the report identifies issues such as checking ballot existence before removal ([L-02]), verifying the existence of a country before proposing exclusion ([L-03]), and checking for the existence of a ballotId before counting votes ([L-04]). Other issues include ensuring values are not smaller than a specified threshold ([L-05]), checking for pool whitelisting ([L-06]), and addressing various non-critical concerns ([N-01] to [N-06]). Remedial actions and mitigations are provided for each issue.

### Summary Table:

| Issue Number | Title                                           | Number of Instances |
|--------------|-------------------------------------------------|---------------------|
| L-01         | Cooldown period added without price change      | 1                   |
| L-02         | Check if ballot exists before removal           | 1                   |
| L-03         | Check if country already exists                 | 1                   |
| L-04         | Check if ballotId exists before counting votes | 3                   |
| L-05         | Check if values are smaller than `pool.dust`    | 3                   |
| L-06         | Check if pool already whitelisted              | 1                   |
| N-01         | Revert with an error message                    | 3                   |
| N-02         | Add else and revert on else                     | 1                   |
| N-03         | Require increment amount to be > 0              | 1                   |
| N-04         | Emit vote updated and vote casted events        | 1                   |
| N-05         | Check for token decimals under 18               | 1                   |
| N-06         | Function requires more comments                | 1                   |
| R-01         | Add msg.sender to proposal created message     | 1                   |
| R-02         | `recoverSalt()` should be called after `unstake` | 1                   |
| R-03         | Emit an event on swap                           | 1                   |
| R-04         | Consider adding remaining amount event          | 1                   |


## [L-01] Cooldown period will be added even if the price is not changed
On numbers greater than 3, the function will have a false cooldown change and a false emit.
### Instances
* [PriceAggregator.sol #61](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L61)
```solidity
else if ( priceFeedNum == 3 )
			priceFeed3 = newPriceFeed;

		priceFeedModificationCooldownExpiration = block.timestamp + priceFeedModificationCooldown;
		emit PriceFeedSet(priceFeedNum, newPriceFeed);
```
### Mitigation
Add a final else statement and revert or return.





## [L-02] Check if ballot exists first before removing to avoid false removes and false emits

### Instancs
* [Proposals.sol #130](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L130)
```solidity
	function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
		{
		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

		Ballot storage ballot = ballots[ballotID];

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



## [L-03] Check if country already exists 
### Instances 
### Instances
* [Proposals.sol #231](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L231)
```solidity
	function proposeCountryExclusion( string calldata country, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

		string memory ballotName = string.concat("exclude:", country );
		return _possiblyCreateProposal( ballotName, BallotType.EXCLUDE_COUNTRY, address(0), 0, country, description );
		}

```



## [L-04] Check if ballotId exists before counting votes
### Instances
* [Proposals.sol #344](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L344)
```solidity
		mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];

		Ballot memory ballot = ballots[ballotID];

```
* [Proposals.sol #357](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L357)
```solidity
		mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];


```

* [Proposals.sol #366](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L366)
```solidity
		mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];


```
## [L-05] Check if any of these values are smaller than `pool.dust`
### Instances
* [Pools.sol #332](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L332)
```solidity
		(uint256 reservesA0, uint256 reservesA1) = getPoolReserves( weth, arbToken2);


```
* [Pools.sol #333](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L333)
```solidity
		(uint256 reservesB0, uint256 reservesB1) = getPoolReserves( arbToken2, arbToken3);


```
* [Pools.sol #334](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L334)
```solidity
		(uint256 reservesC0, uint256 reservesC1) = getPoolReserves( arbToken3, weth);


```

## [L-06] Check if pool is already whitelisted
### Instances
* [PoolsConfig.sol #53]()
```solidity
		_whitelist.add(poolID);

```

## [N-01] Revert with an error message instead of returning 0 value
### Instances
* [Pools.sol #310](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L310)
```solidity
		if ( (reservesWETH < PoolUtils.DUST) || (reservesTokenIn < PoolUtils.DUST) )
			return 0; 

```
* [Pools.sol #315](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L315)
```solidity
		if ( swapAmountInValueInETH <= PoolUtils.DUST )
			return 0;

```
* [Pools.sol #325](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L325)
```solidity
		if ( swapAmountInValueInETH == 0 )
			return 0;

```

## [N-02] Add else and revert on else
Should add an else statement and revert with an error message instead of returning the variable un-initialized with zero value
### Instances
* [Pools.sol #340](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L340)
```solidity
		if (arbitrageAmountIn > 0)
			arbitrageProfit = _arbitrage(arbToken2, arbToken3, arbitrageAmountIn);

```





## [N-03] Require increment amount to be > 0
### Instance 
*[Liquidizer.sol #]()
```solidity
		usdsThatShouldBeBurned += usdsToBurn;

```

## [N-04] Emit vote updated in case of vote update, and vote casted in case of a vote cast
### Instances
* [Proposals.sol #292](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L292)
```solidity
		if ( lastVote.votingPower > 0 )
			_votesCastForBallot[ballotID][lastVote.vote] -= lastVote.votingPower;

		// Update the votes cast for the ballot with the user's current voting power
		_votesCastForBallot[ballotID][vote] += userVotingPower;

		// Remember how the user voted in case they change their vote later
		_lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower );

		emit VoteCast(msg.sender, ballotID, vote, userVotingPower);

```

## [N-05] check for token decimals under 18
Tokens with decimals above 18 may break the contract
### Instances
* [Proposals.sol #162](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L162)
```solidity
		emit ProposalCreated(ballotID, ballotType, ballotName);
	function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID){
+        //require tokens.decimals() <= 18;

        ...
    }

```


## [N-06] Function requires more comments
add comments to describe function behavior, input, and output
### Instances
* [Proposals.sol #81](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L81)
```solidity
function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
```


## [R-01] Add msg.sender to proposal created message
### Instances 
* [Proposals.sol #117](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L117)
```solidity
		emit ProposalCreated(ballotID, ballotType, ballotName);

```

## [R-02] `recoverSalt()` should be called after the `unstake()` method
### Instances
* [Staking.sol #60](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L60)

## [R-03] emit an event on swap
### Instances
* [Pools.sol #366](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L366)


## [R-04] Consider adding the remaining amount or wallet closed event on paying the full amount
### Instances
* [CollateralAndLiquidity.sol #131](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L131)
```solidity
		if ( usdsBorrowedByUsers[msg.sender] == 0 )
			_walletsWithBorrowedUSDS.remove(msg.sender);

		emit RepaidUSDS(msg.sender, amountRepaid);
```

