### Low-01: Users can vote forever till a ballot is finalized. Voting has no deadline before a permissionless finalization call results in vote racing and unfair outcomes.

In Proposals.sol, users can vote techinically forever until a permissionless finalization call. This allows users to race vote and call for finalization against each other for every proposal. Two users might try to front run each other's call to `finalizeBallot()`.

Although technically ok, this might result in a temporary state of vote change to decide the outcome. A person who can submit more gas or is more experienced in front-running might have a better chance of winning a proposal, as supposed to allow all users of various technical proficiency levels to make their votes counted fairly.

```solidity
//src/dao/Proposals.sol
	function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{
...
		//@audit : this only checks whether ballot has been finalized/executed. But it allows vote to be cast any time before a call to `finalizeBallot()`
|>		require( ballot.ballotIsLive, "The specified ballot is not open for voting" );
...
```
(https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L264)

For reference, `ballotIsLive` will only be turned off at the end of `finalizeBallot()` in `markBallotAsFinalized()`. So voting can go on till the final proposal execution.

```solidity
	function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
...
		_allOpenBallots.remove( ballotID );

		ballot.ballotIsLive = false;
...
```
Recommendations:
Consider adding a Voting duration parameter to constraint each proposal to a fixed voting deadline before `finalizeBallot()` is allowed.

### Low-02: Incorrect comment - Contract doesn't allow aidrop participants to vote on country exclusions, in conflict with comments.

In BootstrapBallot.sol, contract comment says `// Allows airdrop participants to vote on whether or not to start up the exchange and which countries should be initially excluded from access`.

Current implementation only allows airdrop participants to vote on starting exchange, but no means to vote on countries.

Comments are incorrect, if the intention is allowing them to vote on country exclusions, this is not implemented.

Recommendations:
Correct comment.


### Low-03: Incorrect assumption of chainlink price feed decimals
In CoreChainlinkFeed.sol, all chainlink price feed is assumed to be in 8 decimals. `// Convert the 8 decimals from the Chainlink price to 18 decimals` This is only true for non-ETH pairs and is fine for fetching collateral prices (WETH, WBTC). However, this will be problematic if `lastChainlinkPrice()` is used to price ETH pairs which will be in 18 decimals.

```solidity
//src/price_feed/CoreChainlinkFeed.sol

function latestChainlinkPrice(
        AggregatorV3Interface chainlinkFeed
    ) public view returns (uint256) {
        int256 price = 0;
...
        // Convert the 8 decimals from the Chainlink price to 18 decimals
        //@audit price will be incorrectly scaled for ETH pairs (in 18 decimals)
 |>     return uint256(price) * 10 ** 10;
```
(https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L58)
Recommendations:
Currently `latestChainlinkPrice()` allows take in any `chainlinkFeed` which might not be Non-eth pair. Consider adding a check to see whether the `chainlinkFeed` address input is only for BTC/USD or ETH/USD. 


### Low-04: Dust and extremely uneven reserve ratio can be established by the first liquidity depositor

When a token is whitelisted by DAO, (WETH, token) and (WBTC, token) will be deployed with zero reserve assets inside. And current impelmentation allows dust liquidity deposit and any reserve ratio for the first depositor.

```solidity
//src/pools/Pools.sol
    function _addLiquidity(
        bytes32 poolID,
        uint256 maxAmount0,
        uint256 maxAmount1,
        uint256 totalLiquidity
    )
...
        if ((reserve0 == 0) || (reserve1 == 0)) {
            // Update the reserves
|>            reserves.reserve0 += uint128(maxAmount0);
|>            reserves.reserve1 += uint128(maxAmount1);

            // Default liquidity will be the addition of both maxAmounts in case one of them is much smaller (has smaller decimals)
|>            return (maxAmount0, maxAmount1, (maxAmount0 + maxAmount1));
        }
...
```
(https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L97-L104)

There are no checks to see of maxAmount0 and maxAmount1 are dust amount, and no check on the ratio of the two when it is the very first deposit.

When the first depositor adds extremely uneven reserves at a dust cost, this allows pool reserve ratios to be manipulated from the beginning. 

Recommendations:
Consider not allowing a dust amount of liquidity adding, and also consider adding a recommended reserve ratio value from the ballot to use as a reference for the initial deposit.


### Low-05: `_userUnstakeIDS` is not updated when unstaking is canceled or claimed. This causes `unstakesForUser` to still retrieve canceled or claimed unstakes as pending

In Staking.sol, a user can unstaked their SALT tokens which starts a delayed process for SALT token claiming. A user can also cancel their unstake request through `cancelUnstake()` which allows them to continuously gain staking rewards. Otherwise, user can claim their staked token when time has passed through `recoverSALT()`. 

The issue is `cancelUnstake()` and `recoverSALT()` don't update users staking status completely. User's `unstakeID` will never be deleted from `_userUnstakeIDs[msg.sender]` array. This will results in user shows up in `unstakesForUser()` view function query as having pending unstakes even though user already canceled or claimed their unstakes. 
(1)
```solidity
//src/staking/Staking.sol
    function cancelUnstake(uint256 unstakeID) external nonReentrant {
...
        //@audit note: this doesn't remove `unstakeID` from _userUnstakeIDs[msg.sender] arrays.
        u.status = UnstakeState.CANCELLED;
        emit UnstakeCancelled(msg.sender, unstakeID);
}
```
(https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L95)
(2)
```solidity
//src/staking/Staking.sol
function recoverSALT( uint256 unstakeID ) external nonReentrant
...
		u.status = UnstakeState.CLAIMED;
```
(https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L108C1-L108C35)

Recommendations:
Consider removing `unstakeID` from `_userUnstakeIDs[msg.sender]` array. Or in `unstakesForUser()` function, filter out `unstakeID` status that is not pending.