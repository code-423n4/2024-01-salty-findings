# QA Report

## Table of Contents

| Issue ID                                                                                                                      | Description                                                                                                       |
| ----------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-proposecountryinclusion-does-not-have-proper-checks-to-ensure-that-a-valid-country-is-proposed-to-be-included) | `proposeCountryInclusion()` does not have proper checks to ensure that a valid country is proposed to be included |
| [QA-02](#qa-02-hardcoding-recipient-while-withdrawing-is-bad-practice)                                                        | Hardcoding recipient while withdrawing is bad practice                                                            |
| [QA-03](#qa-03-protocols-implementation-of-feed-addresses-could-lead-to-a-dos-of-not-less-than-a-month)                       | Protocol's implementation of feed addresses could lead to a DOS of not less than a month                          |
| [QA-04](#qa-04-liquidation-could-be-frontrun)                                                                                 | Liquidation could be frontrun                                                                                     |
| [QA-05](#qa-05-latestrounddata-returns-0-in-any-failure-would-probably-fault-multiple-logics)                                 | `latestRoundData()` returns 0 in any failure would probably fault multiple logics                                 |
| [QA-06](#qa-06-values-are-still-being-stored-in-ether-value)                                                                  | Values are still being stored in ether value                                                                      |
| [QA-07](#qa-07-excludedcountriesupdated-should-be-called-after-a-country-gets-included)                                       | `excludedCountriesUpdated()` should be called after a country gets included                                       |
| [QA-08](#qa-08-possiblycreateproposal-should-be-effectively-reimplemented)                                                    | `_possiblyCreateProposal()` should be effectively reimplemented                                                   |
| [QA-09](#qa-09-btc-feeds-are-planned-to-be-used-for-wbtc-which-would-cause-a-massive-pricing-error-if-wbtc-ever-depegs)       | Btc feeds are planned to be used for wbtc which would cause a massive pricing error if wbtc ever depegs           |
| [QA-10](#qa-10-tokenwhitelistingballotwiththemostvotes-should-consider-more-edge-cases)                                       | `tokenWhitelistingBallotWithTheMostVotes()` should consider more edge cases                                       |

## QA-01 `proposeCountryInclusion()` does not have proper checks to ensure that a valid country is proposed to be included

### Proof of Concept

Take a look at [Proposals.sol#L224-L232](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/Proposals.sol#L224-L232)

```solidity
	function proposeCountryInclusion( string calldata country, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );

		string memory ballotName = string.concat("include:", country );
		return _possiblyCreateProposal( ballotName, BallotType.INCLUDE_COUNTRY, address(0), 0, country, description );
		}


```

Now consider this:

```js
// Example of a vulnerable input
string calldata country = "A,";  // Contains a comma
string calldata description = "Some description";

// Calling the function with the vulnerable input
uint256 ballotID = proposeCountryInclusion(country, description);
```

In the above example, we have provided an input `country` with the value "A,", which includes a comma. According to the current validation check, this input would be accepted as a valid ISO 3166 Alpha-2 country code because it has a length of 2 characters. However, it violates the expected format of a valid country code, cause from [here](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) we can see that all valid codes are to be made up of two alphabetical characters from `AA..=ZZ`

### Foundry test case

Apply these modifications to the already existing test here: https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/tests/Proposals.t.sol#L1356-L1372

```diff
    // A unit test for the proposeCountryInclusion and proposeCountryExclusion functions to verify they don't allow an empty country name.
    function testProposeCountryInclusionExclusionEmptyName() public {
-        string memory emptyCountryName = "";
+        string memory emptyCountryName = "A,";

        // Proposing country inclusion with empty country name should fail
        vm.startPrank(alice);
        staking.stakeSALT(1000 ether);
-        vm.expectRevert("Country must be an ISO 3166 Alpha-2 Code");
// no reversion would occur even when code is not a valid Alpha-2 country code
        proposals.proposeCountryInclusion(emptyCountryName, "description");
        vm.stopPrank();

        // Proposing country exclusion with empty country name should fail
        vm.startPrank(alice);
-        vm.expectRevert("Country must be an ISO 3166 Alpha-2 Code");
        proposals.proposeCountryExclusion(emptyCountryName, "description");
        vm.stopPrank();
    }
```

### Impact

The input validation in the code does not sufficiently verify the format of the country code, potentially allowing invalid inputs to be accepted. This leads to unexpected behavior in the smart contract and compromises its functionality. Specifically, the current validation check only verifies the length of the input string and does not ensure that it consists of valid uppercase alphabetic characters.

### Recommended Mitigation Steps

To enhance the input validation and ensure that the country code adheres to the ISO 3166 Alpha-2 code format, consider verifying that both characters in the input string are uppercase letters (A-Z):

```solidity
require(
    (bytes(country)[0] >= bytes("A")[0] && bytes(country)[0] <= bytes("Z")[0]) &&
    (bytes(country)[1] >= bytes("A")[0] && bytes(country)[1] <= bytes("Z")[0]),
    "Country code must consist of two uppercase letters"
);
```

## QA-02 Hardcoding recipient while withdrawing is bad practice

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/pools/Pools.sol#L218-L231

```solidity

    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}

```

As seen, this function is used to withdraw tokens that were previously deposited for users, issue here is thatr the function hardcodes the recipient and does not allow the `msg.sender` to provide a fresh address to possibly send these tokens, now in a case where tokens like USDC and USDT are whitelisted (i.e tokens that employ blacklisting mechanisms) and `msg.sender` now becomes blacklisted, then users funds are stuck in the contract.

### Impact

Potential funds stuckage for users, depending on whitelisting of tokens that employ this functionality.

### Recommended Mitigation Steps

Accept a fresh address for the `withdraw()` function and if user doesn't provide this address then funds xan just be sent to the `msg.sender`.

## QA-03 Protocol's implementation of feed addresses could lead to a DOS of not less than a month

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/price_feed/PriceAggregator.sol#L31-L35

```solidity
	// The required cooldown between calls to setPriceFeed.
	// Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
	// Range: 30 to 45 days with an adjustment of 5 days
    //@audit
	uint256 public priceFeedModificationCooldown = 35 days;

```

And https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/price_feed/PriceAggregator.sol#L47-L52

```solidity
	function setPriceFeed( uint256 priceFeedNum, IPriceFeed newPriceFeed ) public onlyOwner
		{
		// If the required cooldown is not met, simply return without reverting so that the original proposal can be finalized and new setPriceFeed proposals can be made.
		if ( block.timestamp < priceFeedModificationCooldownExpiration )
			return;

```

As seen this duration "priceFeedModificationCooldownExpiration" is used as a period to evaluate the performance of the recently updated PriceFeed before further updates are made, now issue with this is that, when it comes to chainlink feeds they could getr deprecated and as such their addresses could even change which would cause calls to query the prices to even revert.

### Impact

Intended design could actually cause a DOS of **at least 30 days**, i.e an instance where the stored address for the chainlink feed is inacessible and it can't be changed cause protocol needs to wait for a range of 30..45 days, cause [these queries](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/price_feed/PriceAggregator.sol#L175-L196) would always revert at an attempt to get the price from the second pricefeed

### Recommended Mitigation Steps

Have a side stepper for emergency situtations to set the chainlink feeds for scenarios like explained in the report.

## QA-04 Liquidation could be frontrun

### Proof of Concept

When a position is no longer sufficiently collateralized, the `liquidateUser() `function can be called to liquidate the position and earn a 5% bonus (current default value). However, when someone finds a liquidation opportunity and calls this function, anyone can frontrun it and execute the liquidation first to get the bonus reward.
Take a look at https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/stable/CollateralAndLiquidity.sol#L140-L191

```solidity
	function liquidateUser( address wallet ) external nonReentrant
		{
		require( wallet != msg.sender, "Cannot liquidate self" );

		// First, make sure that the user's collateral ratio is below the required level
		require( canUserBeLiquidated(wallet), "User cannot be liquidated" );

		uint256 userCollateralAmount = userShareForPool( wallet, collateralPoolID );

		// Withdraw the liquidated collateral from the liquidity pool.
		// The liquidity is owned by this contract so when it is withdrawn it will be reclaimed by this contract.
		(uint256 reclaimedWBTC, uint256 reclaimedWETH) = pools.removeLiquidity(wbtc, weth, userCollateralAmount, 0, 0, totalShares[collateralPoolID] );

		// Decrease the user's share of collateral as it has been liquidated and they no longer have it.
		_decreaseUserShare( wallet, collateralPoolID, userCollateralAmount, true );

		// The caller receives a default 5% of the value of the liquidated collateral.
		uint256 rewardPercent = stableConfig.rewardPercentForCallingLiquidation();

		uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
		uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;

		// Make sure the value of the rewardAmount is not excessive
		uint256 rewardValue = underlyingTokenValueInUSD( rewardedWBTC, rewardedWETH ); // in 18 decimals
		uint256 maxRewardValue = stableConfig.maxRewardValueForCallingLiquidation(); // 18 decimals
		if ( rewardValue > maxRewardValue )
			{
			rewardedWBTC = (rewardedWBTC * maxRewardValue) / rewardValue;
			rewardedWETH = (rewardedWETH * maxRewardValue) / rewardValue;
			}

		// Reward the caller
		wbtc.safeTransfer( msg.sender, rewardedWBTC );
		weth.safeTransfer( msg.sender, rewardedWETH );

		// Send the remaining WBTC and WETH to the Liquidizer contract so that the tokens can be converted to USDS and burned (on Liquidizer.performUpkeep)
		wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );
		weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );

		// Have the Liquidizer contract remember the amount of USDS that will need to be burned.
		uint256 originallyBorrowedUSDS = usdsBorrowedByUsers[wallet];
		liquidizer.incrementBurnableUSDS(originallyBorrowedUSDS);

		// Clear the borrowedUSDS for the user who was liquidated so that they can simply keep the USDS they previously borrowed.
		usdsBorrowedByUsers[wallet] = 0;
		_walletsWithBorrowedUSDS.remove(wallet);

		emit Liquidation(msg.sender, wallet, reclaimedWBTC, reclaimedWETH, originallyBorrowedUSDS);
		}



```

### Impact

On the long run, there might no longer be an incentive to liquidate bad positions because the liquidation rewards can be stolen by frontrunning.

### Recommended Mitigation Steps

Implement a Commit-Reveal mechanism or incentivize users to use Flashbots when calling the `liquidateUser()` function.

## QA-05 `latestRoundData()` returns 0 in any failure would probably fault multiple logics

### Impact

Faulting of multiple pricing logic, this is cause even if protocol uses two different sources of pricing having another one being faulty and the returned price being zero could end up causing protocol to even unfairly liquidate a user and what not.

### Proof of Concept

Looking at the pricing logic used in protocol for chainlink we can see that it returns 0 if it encounters any error.

### Recommended Mitigation Steps

Reimplement what's returned if the querying fails for whatever reason.

## QA-06 Values are still being stored in ether value

### Proof of Concept

Multiple instances of values being stored in ether value to aid with testing that hasn't been updated, for example the `bootstrappingRewards` from the DAOConfig is to be in direct relation with the amount of SALT tokens before whitelisting,[ i.e by multiplying it by 2](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/DAO.sol#L247), issue now is that this value is stored in ether value

### Recommended Mitigation Steps

Store values how they are to be stored.

## QA-07 `excludedCountriesUpdated()` should be called after a country gets included

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/DAO.sol#L184-L202 from `_executeApproval()`

```solidity

		else if ( ballot.ballotType == BallotType.INCLUDE_COUNTRY )
			{
			excludedCountries[ ballot.string1 ] = false;

			emit GeoExclusionUpdated(ballot.string1, false, exchangeConfig.accessManager().geoVersion());
			}

		else if ( ballot.ballotType == BallotType.EXCLUDE_COUNTRY )
			{
			excludedCountries[ ballot.string1 ] = true;

			// If the AccessManager doesn't implement excludedCountriesUpdated, this will revert and countries will not be able to be excluded until the AccessManager is working properly.
			exchangeConfig.accessManager().excludedCountriesUpdated();

			emit GeoExclusionUpdated(ballot.string1, true, exchangeConfig.accessManager().geoVersion());
			}


```

As we can see where as whenever a country gets excluded the `excludedCountriesUpdated` gets called on the access manager this is not done for the inclusion of countries.

### Impact

Uneven code implementation.

### Recommended Mitigation Steps

Add this line: `			exchangeConfig.accessManager().excludedCountriesUpdated();` to the instance pertaining inclusion of countries

## QA-08 `_possiblyCreateProposal()` should be effectively reimplemented

### Impact

The `_possiblyCreateProposal` function appears to have an issue related to the calculation of `requiredXSalt` when certain conditions are met. Specifically, when the total staked shares (`totalStaked`) is less than a certain threshold, `requiredXSalt` can become zero, leading to the failure of proposals. This issue affects the ability of users to create proposals, potentially preventing valid proposals from being submitted and impacting the governance process.

### Proof of Concept

Consider the following scenario:

- The `daoConfig.requiredProposalPercentStakeTimes1000` is set to its maximum value (e.g., 2000).
- The `totalStaked` is less than 50 shares, such as 49 shares.

In this case, when calculating `requiredXSalt`:

```
uint256 requiredXSalt = (totalStaked * daoConfig.requiredProposalPercentStakeTimes1000()) / (100 * 1000);
```

The result will be zero because the expression `(totalStaked * daoConfig.requiredProposalPercentStakeTimes1000())` will round down to zero since `totalStaked` is less than 50 shares. This leads to `requiredXSalt` being zero, which is contrary to the intended behavior where users should have a minimum `requiredXSalt` to create proposals.

### Recommended Mitigation Steps

Reimplement the `_possiblyCreateProposal` function to take this into account.

## QA-09 Btc feeds are planned to be used for wbtc which would cause a massive pricing error if wbtc ever depegs

### Proof of Concept

Looking at protocol we can see that it uses the btc/usd feed instead of the wbtc/usd one

### Impact

A massive pricing error if wbtc ever depegs

### Recommended Mitigation Steps

Use chainlink's `wbtc/usd` feed instead of

## QA-10 `tokenWhitelistingBallotWithTheMostVotes()` should consider more edge cases

### Impact

The `tokenWhitelistingBallotWithTheMostVotes` function is designed to return the `ballotID` of the whitelisting ballot that currently has the most "yes" votes. However, there is an issue in the code that can lead to an edge case where two or more ballots have the same number of "yes" votes, but only one of them is considered, potentially leading to unfair or unexpected results.

Unfair/Unexpected result in the sense that if a call to fialize the whitelisting for this tokens gets called, i.e [DAO.sol#L235-L236](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/DAO.sol#L235-L236), it would revert for an instance of a token with the shared `mostVotes` in as much as it doesn't come first in the iteration

### Proof of Concept

Take a look at [Proposals.sol#L419-L444](https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/dao/Proposals.sol#L419-L444)

```solidity
	function tokenWhitelistingBallotWithTheMostVotes() external view returns (uint256)
		{
		uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN);

		uint256 bestID = 0;
		uint256 mostYes = 0;
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
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

		return bestID;
		}



```

Now consider a scenario where two whitelisting ballots have the following vote totals:

- Ballot 2: 50 "yes" votes, 25 "no" votes
- Ballot 4: 50 "yes" votes, 20 "no" vote
  _Assuming a quorum of 70 votes in total_

According to the current code, the `tokenWhitelistingBallotWithTheMostVotes` function would select Ballot 2 as the one with the most "yes" votes because it is the first one encountered that meets the conditions. Ballot 4, with an equal number of "yes" votes, would not be considered.

### Recommended Mitigation Steps

To address the issue where multiple whitelisting ballots have the same number of "yes" votes, and only one is considered, we can implement a modification to ensure that all ballots with the most "yes" votes are considered, the function could be reimplemented to then return an array of `ballotID` values for all valid whitelisting ballots with the most "yes" votes. This ensures that all deserving ballots are considered, eliminating the edge case where only one is selected when multiple have the same "yes" vote count.
