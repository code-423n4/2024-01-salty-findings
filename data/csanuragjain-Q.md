## No_change will be chosen incorrectly

Lines of code 
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L372-L380

Impact
In case of tie between Increase and Decrease, No change will be chosen

Proof of Concept
1. The winning parameter is chosen using:

```
function winningParameterVote( uint256 ballotID ) external view returns (Vote)
		{
		mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];

		uint256 increaseTotal = votes[Vote.INCREASE];
		uint256 decreaseTotal = votes[Vote.DECREASE];
		uint256 noChangeTotal = votes[Vote.NO_CHANGE];

		if ( increaseTotal > decreaseTotal )
		if ( increaseTotal > noChangeTotal )
			return Vote.INCREASE;

		if ( decreaseTotal > increaseTotal )
		if ( decreaseTotal > noChangeTotal )
			return Vote.DECREASE;

		return Vote.NO_CHANGE;
		}
```

2. Let's say INCREASE and DECREASE are both 100 and NO_CHANGE is 0. In this case below flow happens:

```
increaseTotal > decreaseTotal which is 100>100 which is false
decreaseTotal > increaseTotal which is 100>100 which is false
So Vote.NO_CHANGE is returned
```

3. So in this case No_change is selected even though noone voted for NO_CHANGE and all votes were for Increase and Decrease

Recommendation
Document this behavior so that Users are aware

## Unfair Ballot selection

Lines of code
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L431

Impact
Ballot with highest yes vote won't be selected right away, for token whitelisting under certain circumstances. Ballot which are created early are given preference which might not be fair always

Proof of Concept
1. Lets say 3 ballots exists for token whitelisting
2. Below are the param used

```md
quorum=100

Ballot 1:
yesVotes=80
noVotes=30

Ballot 2:
yesVotes=80
noVotes=20

Ballot 3:
yesVotes=70
noVotes=30

```

3. Now once Ballots are finalized for whitelisting, below call sequence occurs:

```
_finalizeTokenWhitelisting -> proposals.tokenWhitelistingBallotWithTheMostVotes();
```

4. `tokenWhitelistingBallotWithTheMostVotes` will iterate through each ballot and find the one with most yesVotes like below:

```solidity
if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
				{
				bestID = ballotID;
				mostYes = yesTotal;
				}
```

5. Below is the run sequence:

```
Ballot 1:

(yesTotal + noTotal) >= quorum is true since 80+30>=100
yesTotal > noTotal is true since 80>30
mostYes=80
bestID=Ballot 1

Ballot 2:

(yesTotal + noTotal) >= quorum is true since 80+20>=100
yesTotal > noTotal is true since 80>20
mostYes=80
yesTotal > mostYes is false since 80>80 is false
bestID remains Ballot 1
```

6. As we can see that even when both Ballot have equal yesVotes, Ballot1 gets chosen, meaning Ballot1 asset gets whitelisted which is unfair for Ballot2 voters

```
uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

			// All tokens are paired with both WBTC and WETH, so whitelist both pairings
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
```

Recommended Mitigation Steps
This behavior can be documented