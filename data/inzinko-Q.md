## [L-01] Users can still vote after the ballot expiry is over

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L259C2-L293C4

If the Ballot end time has been completed, users are not supposed to be able to cast vote again, but the cast vote does not check that the ballot end time has reached
```solidity
function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{
		Ballot memory ballot = ballots[ballotID];

		// Require that the ballot is actually live
		require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

//@audit This check here that asks if the ballot is live or not does not prove that the ballot end time has been completed, because that value is only updated if the ballot has been finalized

...more code
		}
```

- Only on finalizeBallot does ballotIsLive gets updated
This means that before finalize ballot is called users can still cast their vote

Mitigation 
Implement a check to see if The ballot duration has ended 

## [L-02] Different amount of SALT rewards from Protocol Owned Liquidity to be burnt from the Docs and the Code Implementation

In the Docs of the project, it says that 45% of SALT rewards must be burnt, but this value is different in the code 

the docs as shown below 

> It generates yield for the DAO in the form of SALT.  By default 45% of this generated SALT is burned.

https://docs.salty.io/decentralization/protocol-owned-liquidity

But in the code of the `DAOConfig` The Value is 50%

as shown below

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L26C2-L28C49

```solidity
// For rewards distributed to the DAO, the percentage of SALT that is burned with the remaining staying in the DAO for later use.
	// Range: 25% to 75% with an adjustment of 5%
    uint256 public percentPolRewardsBurned = 50;
```
## [L-03] Pools contract can contain Unwhitelisted Tokens 

In the `Pools::deposit` users can deposit unwhitelisted tokens because the function does not check if the tokens that are about to be sent are whitelisted tokens, it just allow them to deposit them as shown below 

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L205C2-L215C4

```solidity
function deposit( IERC20 token, uint256 amount ) external nonReentrant
		{
        require( amount > PoolUtils.DUST, "Deposit amount too small");

		_userDeposits[msg.sender][token] += amount;

		// Transfer the tokens from the sender - only tokens without fees should be whitelsited on the DEX
		token.safeTransferFrom(msg.sender, address(this), amount );
//@audit No check for Non-Whitelisted token 

		emit TokenDeposit(msg.sender, token, amount);
		}
```
This Goes against the rules of the protocol, as only whitelisted tokens should be present in the Pool and any other contract 