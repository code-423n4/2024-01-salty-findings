# [L01]: Users with > 0.5% of staked salt can prevent confirmation proposals from proceeding via name collision.

## Vulnerability Details
For proposals such as `SET_CONTRACT` or `SET_WEBSITE_URL`, the DAO creates a second confirmation proposal for users to vote on after the initial proposal passes.

[DAO.sol#L202](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L202)
```solidity
		else if ( ballot.ballotType == BallotType.SET_CONTRACT )
			proposals.createConfirmationProposal( string.concat(ballot.ballotName, "_confirm"), BallotType.CONFIRM_SET_CONTRACT, ballot.address1, "", ballot.description );

		// Once an initial setWebsiteURL proposal passes, it automatically starts a second confirmation ballot (to prevent last minute approvals)
		else if ( ballot.ballotType == BallotType.SET_WEBSITE_URL )
			proposals.createConfirmationProposal( string.concat(ballot.ballotName, "_confirm"), BallotType.CONFIRM_SET_WEBSITE_URL, address(0), ballot.string1, ballot.description );
```
When a confirmation proposal is created, the ballot name of the confirmation proposal will be set to the original ballot name + `_confirm`. 

The DAO will proceed to then create a confirmation proposal using `createConfirmationProposal` which calls `_possiblyCreateProposal`

[Proposals.sol#L81C1-L128C1](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L81C1-L128C1)
```solidity
	function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
		{
		require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

		// The DAO can create confirmation proposals which won't have the below requirements
		if ( msg.sender != address(exchangeConfig.dao() ) )
			{
			// Make sure that the sender has the minimum amount of xSALT required to make the proposal
			uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

			require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

			uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
			require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );

			// Make sure that the user doesn't already have an active proposal
			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
			}

		// Make sure that a proposal of the same name is not already open for the ballot
=>		require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
=>		require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );

		uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration();

		// Add the new Ballot to storage
		ballotID = nextBallotID++;
		ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime );
		openBallotsByName[ballotName] = ballotID;
		_allOpenBallots.add( ballotID );

		// Remember that the user made a proposal
		_userHasActiveProposal[msg.sender] = true;
		_usersThatProposedBallots[ballotID] = msg.sender;

		emit ProposalCreated(ballotID, ballotType, ballotName);
		}


	// Create a confirmation proposal from the DAO
	function createConfirmationProposal( string calldata ballotName, BallotType ballotType, address address1, string calldata string1, string calldata description ) external returns (uint256 ballotID)
		{
		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );

=>		return _possiblyCreateProposal( ballotName, ballotType, address1, 0, string1, description );
		}

```

The problem in `_possiblyCreateProposal` is that it checks:

1) whether there already is an open proposal with the same name 

2) whether there already is an open proposal with the same name + "_confirm". [not important]

and reverts if so.

However, the behaviour in 1 can be used against a protocol by creating a new `SET_CONTRACT` or `SET_WEBSITE_URL` proposal with a ballot name of an active `SET_CONTRACT` or `SET_WEBSITE_URL` proposal with the `_confirm` suffix appended, since creating these proposals do not check for the `_confirm` suffix.

[Proposals.sol#L240C1-L255C4](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L240C1-L255C4)
```solidity
	function proposeSetContractAddress( string calldata contractName, address newAddress, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( newAddress != address(0), "Proposed address cannot be address(0)" );

		string memory ballotName = string.concat("setContract:", contractName );
		return _possiblyCreateProposal( ballotName, BallotType.SET_CONTRACT, newAddress, 0, "", description );
		}


	function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );

		string memory ballotName = string.concat("setURL:", newWebsiteURL );
		return _possiblyCreateProposal( ballotName, BallotType.SET_WEBSITE_URL, address(0), 0, newWebsiteURL, description );
		}
```

If the initial `SET_CONTRACT` or `SET_WEBSITE_URL` proposal passes, the DAO cannot proceed with the confirmation proposal because an attacker that is unhappy with the initial `SET_CONTRACT` or `SET_WEBSITE_URL` proposal has already created  a proposal with a name colliding with the confirmation proposal to be set.

Though the confirmation proposal can be created after an attacker's proposal is finalised, if the attacker is fast they can create more of such proposal to further prevent creation of the confirmation proposal.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Check for `_confirm` suffix in the ballot name whenever `proposeSetContractAddress` or `proposeWebsiteUpdate` is called