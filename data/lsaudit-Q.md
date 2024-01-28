# [1] Percentage of rewards sent to the initial team is hardcoded in the contract, thus it's not possible to change it in the future

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L340)
```
		// Send 10% of the rewards to the initial team
		uint256 amountToSendToTeam = claimedSALT / 10;
```

The percentage of rewards which will be sent to the initial team is set to 10% and hard-coded into the contract. It's not possible to change it in the future. Consider implementing a way of changing this value in the future and do not hard-code it in the protocol.


# [2] Mixing tabs and white-space in code-formatting

Multiple of contracts does not obey the best coding standard in the context of tabulations and white-spaces. Tabs and white-spaces are mixed.
This issue occurs basically in every contract. For the QA report readability, we've marked down only two examples of this issue. Nonetheless, this issue affects every contract in scope.

[File: InitialDistribution](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L52-L53)
```
    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );  // @audit: 4 whitespaces + tab
		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" ); // // @audit: 2 tabs
```

[File: USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L22-L25)
```
	constructor()
	ERC20( "USDS", "USDS" )
		{ // @audit: 2 tabs
        }  // @audit: 8 whitespaces

```

[File: CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L30-L36)
```
	using SafeERC20 for IERC20;  // @audit: 1 tab
	using SafeERC20 for IUSDS; // @audit: 1 tab
    using EnumerableSet for EnumerableSet.AddressSet; // @audit: 4 whitespaces

    IStableConfig immutable public stableConfig; // @audit: 4 whitespaces
	IPriceAggregator immutable public priceAggregator; // @audit: 1 tab
    IUSDS immutable public usds; // @audit: 4 whitespaces
```

# [3] The same wallet can be authorized multiple of times in `Airdrop.sol`

Function `authorizeWallet()` utilizez OZ's EnumerableSet implementation. If the set already contains the value which protocol tries to add - function `add()` will return `false` (instead of reverting). This implies that function `authorizeWallet()` from `Airdrop.sol` can be called multiple of times, with the same wallet address.
Basically, it won't pose any security risk (since the `wallet` would had already been added to `_authorizedUsers`) - thus this issue is only reported in the QA report.

A good security practice is to verify if the value is already in set and revert if it is. This could be achieved by changing below code:

[File: Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L51)
```
_authorizedUsers.add(wallet);
```

to:

```
require(_authorizedUsers.add(wallet), "already added");
```

# [4] Function `vote()` does not verify the contract address, meaning it might be possible to vote multiple of times on the same chain

[File: BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L53)
```
		bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
		require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );
```

The `messageHash` contains only `block.chanid` and `msg.sender`. If the contract would be re-deployed to the different address (or when someone else would decide to use the fork of the protocol's implementation and deploy it to the different address) - then it would be possible to re-use `messageHash`. 
It's good security practice to utilize `address(this)` when calculating `messageHash`, so that it won't be possible to re-use the same `messageHash` on the different contract addresses.



# [5] Lack of address(0) check in `setPriceFeed()` in `PriceAggregator.sol`

[File: PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L53)
```
		if ( priceFeedNum == 1 )
			priceFeed1 = newPriceFeed;
		else if ( priceFeedNum == 2 )
			priceFeed2 = newPriceFeed;
		else if ( priceFeedNum == 3 )
			priceFeed3 = newPriceFeed;
```

Function `setPriceFeed()` does not perform any validation for `address(0)`. Whenever `newPriceFeed` parameter is passed as `address(0)`, function will set `priceFeed1` (or `priceFeed2`, `priceFeed3`) to `address(0)`. Since this function is called by only owner (`onlyOwner` modifier), the severity has been reduced to Low and included in QA report. Nonetheless, it's good practice to verify if address is not `address(0)`, whenever we set new price feed.

Our recommendation is to add additional check:

```
require(newPriceFeed != address(0), "new price feed cannot be address(0)");
```

# [6]  Setters does not verify if the value was really changed

[File: ExchangeConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ExchangeConfig.sol#L62)
```
	function setAccessManager( IAccessManager _accessManager ) external onlyOwner
		{
		require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" );

		accessManager = _accessManager;

	    emit AccessManagerSet(_accessManager);
		}
```

Whenever we update/set a new value of some state variable, it's a good practice to make sure that, that value is being indeed updated.
In the current implementation of `setAccessManager()`, even when the `_accessManager` won't be changed, function will still emit an `AccessManagerSet()` event - which might be misleading to the end-user.
E.g. let's assume that `_accessManager = 0x123`. Calling `setAccessManager(0x123)` won't update/set anything new, but `AccessManagerSet()` will still be called.

Our recommendation is to add additional `require` check, to verify, that provided values are indeed different than the current ones:

```
require(address(_accessManager) != address(accessManager ), "Nothing to update");
```


# [7] Dust should be linked to tokens based on their decimals

[File: PoolUtils.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L11)
```
	// Token reserves less than dust are treated as if they don't exist at all.
	// With the 18 decimals that are used for most tokens, DUST has a value of 0.0000000000000001
	uint256 constant public DUST = 100;
```

The `DUST` value is set to `100`. While this is a very small amount of 18 decimals tokens, it might not be good enough for tokens with smaller decimals amount.
Protocol does not explicitly states what kind of tokens will be used. There's no assumptions that tokens with, e.g. 8 decimals won't be used. For that tokens, the `DUST` will be defined as `1e-06`. This is still small amount, but, it's extensively higher than `1e-16`.
We did not report this issue as `Medium`, because we believe that it's up to developers to choose if `1e-6` dust for 8 decimals token is acceptable amount of `DUST`.
Nonetheless, this problem could be resolved by having additional mapping, which links the tokens to the acceptable DUST amount for that token. E.g. for tokens with 8 decimals, DUST can be smaller.


# [8] `constructor()` in `ManagedWallet.sol` allows to set wallets' addresses to `address(0)`.

Since this issue occurs in the `constructor()`, the severity of this issue was evaluated as Low, thus this issue is being reported in the QA report.

[File: ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L31)
```
	constructor( address _mainWallet, address _confirmationWallet)
		{
		mainWallet = _mainWallet;
		confirmationWallet = _confirmationWallet;
```

Both `mainWallet` and `confirmationWallet` can be set to `address(0)`, since `constructor()` does not verify if parameters `_mainWallet` and `_confirmationWallet` are not `address(0)`.


# [9] It's possible to call `setContracts()` in `ExchangeCOnfig.sol` multiple of times in some rare conditions

[File: ExchangeConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ExchangeConfig.sol#L47)
```
// setContracts can only be be called once - and is called at deployment time.
	function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
		{
		// setContracts is only called once (on deployment)
		require( address(dao) == address(0), "setContracts can only be called once" );

		dao = _dao;
```

According to the comments, function `setContracts()` should be called only once. This invariant, is however broken, when `_dao` is set to `address(0)`.

Function `setContracts()` does not check if parameter `_dao` is not `address(0)`. This means, that it's possible to call `setContracts()` multiple of times by passing `_dao` as `address(0)`.

When function `setContracts()` is called for the first time with `_dao` set to `address(0)` - `dao` will still point out to `address(0)`. This means, that it would be possible to call this function once again - even though, it should not be possible.

Due to very rare likelihood (only owner needs to call `setContracts()` with `_dao` set to `address(0)`) - this issue has been reported in QA reports only as bad coding practice.

Nonetheless, our recommendation is to add additional check which requires that `_dao` cannot be `address(0)`: `require(_dao != address(0), "dao cannot be addr(0)")`.




# [10] Typo in event name

Event names should start with upper-case letters.

[File: PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L16)
```
event maximumInternalSwapPercentTimes1000Changed(uint256 newMaxSwap);
```

`maximumInternalSwapPercentTimes1000Changed` should be changed to `MaximumInternalSwapPercentTimes1000Changed`.


# [11] Use predefined constants in comments

[File: PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L60)
```
// Returns uint64.max in the event of failed lookup
```

The code uses previously defined constant `INVALID_POOL_ID`. It will be much clearer to use this constant in the comment section, instead of the hard-coded value:

```
// Returns INVALID_POOL_ID (uint64.max) in the event of failed lookup
```


# [12] Incorrect comment in `Pools.sol`

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L270-L271)
```
		// Make sure that the reserves after swap are above DUST
        require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap");
```

Since `>=` operator is being used, the comment should be: `Make sure that the reserves after swap are not below DUST`.


# [13] Stick to one coding-style during function calls

Good coding style defines calling function syntax as: `functionName(parameter)`. However, across multiple of contracts, we have spot a redundant white-space before and/or after the parentheses: `functionName( parameter)`, `functionName(parameter )`, `functionName( parameter )`.
It's a good coding practice to stick to one style in the syntax.

Across every contract, multiple of function calls are being used. E.g., in multiple of places, there's a whitespace after opening parentheses, while in other - the whitespace is missing: `A(test)`, `A( test )`. Having different coding-styles decreases code readability.

This is an overall issue across every contract. Since this is juts a minor issue which does not affect the contract's security at all (it affects only the code quality) - for the report's clarity, we have decided to point out only one example of this issue. Please keep in mid that this problem occurs basically in every contract.

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L332)
```
332: (uint256 reservesA0, uint256 reservesA1) = getPoolReserves( weth, arbToken2);  // @audit: A( test)
350: swapAmountOut = _adjustReservesForSwap( swapTokenIn, swapTokenOut, swapAmountIn ); // @audit: A( test )
411: (bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB); // @audit: A(test)
```



# [14] Use white-space in mappings' syntax

Using white-spaces before and after `=>` in the mappings improves the code readability.

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L369)
```
mapping(IERC20=>uint256) storage userDeposits = _userDeposits[msg.sender];
```

should be changed to:

```
mapping(IERC20 => uint256) storage userDeposits = _userDeposits[msg.sender];
```


Other instances of this issue:

```
./src/dao/DAO.sol:67:   mapping(string=>bool) public excludedCountries;
./src/dao/Proposals.sol:38:     mapping(string=>uint256) public openBallotsByName;
./src/dao/Proposals.sol:41:     mapping(uint256=>Ballot) public ballots;
./src/dao/Proposals.sol:51:     mapping(uint256=>mapping(Vote=>uint256)) private _votesCastForBallot;
./src/dao/Proposals.sol:55:     mapping(uint256=>mapping(address=>UserVote)) private _lastUserVoteForBallot;
./src/dao/Proposals.sol:59:     mapping(address=>bool) private _userHasActiveProposal;
./src/dao/Proposals.sol:63:     mapping(uint256=>address) private _usersThatProposedBallots;
./src/dao/Proposals.sol:344:            mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];
./src/dao/Proposals.sol:357:            mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];
./src/dao/Proposals.sol:366:            mapping(Vote=>uint256) storage votes = _votesCastForBallot[ballotID];
./src/launch/Airdrop.sol:29:    mapping(address=>bool) public claimed;
./src/launch/BootstrapBallot.sol:25:    mapping(address=>bool) public hasVoted;
./src/pools/Pools.sol:46:       mapping(bytes32=>PoolReserves) private _poolReserves;
./src/pools/Pools.sol:49:       mapping(address=>mapping(IERC20=>uint256)) private _userDeposits;
./src/pools/Pools.sol:369:              mapping(IERC20=>uint256) storage userDeposits = _userDeposits[msg.sender];
./src/pools/PoolsConfig.sol:32: mapping(bytes32=>TokenPair) public underlyingPoolTokens;
./src/pools/PoolStats.sol:21:   mapping(bytes32=>uint256) public _arbitrageProfits;
./src/pools/PoolStats.sol:24:   mapping(bytes32=>ArbitrageIndicies) public _arbitrageIndicies;
./src/rewards/RewardsEmitter.sol:35:    mapping(bytes32=>uint256) public pendingRewards;
./src/stable/CollateralAndLiquidity.sol:49:    mapping(address=>uint256) public usdsBorrowedByUsers;
./src/staking/Staking.sol:29:    mapping(uint256=>Unstake) private _unstakesByID;
./src/staking/StakingRewards.sol:36:    mapping(address=>mapping(bytes32=>UserShareInfo)) private _userShareInfo;
./src/staking/StakingRewards.sol:39:    mapping(bytes32=>uint256) public totalRewards;
./src/staking/StakingRewards.sol:42:    mapping(bytes32=>uint256) public totalShares;
./src/staking/StakingRewards.sol:149:           mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[msg.sender];
./src/staking/StakingRewards.sol:294:           mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[wallet];
```


# [15] Document constant numbers used in calculations

[File: PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L142)
```
if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
```

Above code does not contain any comment which explains why we're multiplying by `100000`. Since this is a percent and `Times1000`, we should multiply by `100 * 1000`, which is indeed `100000`. However, using this number without any explanation in the code-base might be very misleading. Our recommendation is to document every non-obvious constant-numbers.


# [16] Functions from `*Config.sol` emits event even when nothing is changed

A lot of functions in `*Config.sol` files implement functions which are defined as below:

```
if (increase) 
{
	if (check if in range) {
		do something
	}
}
else {
	if (check if in range) {
		do something
	}
}

emit someEvent();
```

This basically means that emit is emitted no matter if the action was taken or not.
E.g., let's take a look at `changeBallotDuration()` function:

[File: DAOConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L115)
```
	function changeBallotDuration(bool increase) external onlyOwner
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

When `ballotMinimumDuration` is set to 14 days (we would need to call `changeBallotDuration(true)` a few times to achieve this state), then calling `changeBallotDuration(true)` one more time won't update `ballotMinimumDuration`  (because `14 < 14` is `false` and we won't enter conditional branch at line 119). Nonetheless, the `BallotDurationChanged()` event will still be emitted.

When nothing is changed, function should not emit an event. The event should be moved to the `if` condition, so that it will be only executed when some configuration would be updated.

The same issue occurs in almost every functions in every `*Config.sol` files:

* `DAOConfig.sol`
Functions: `changeBootstrappingRewards(), changePercentPolRewardsBurned(), changeBaseBallotQuorumPercent(), changeBallotDuration(), changeRequiredProposalPercentStake(), changeMaxPendingTokensForWhitelisting(), changeArbitrageProfitsPercentPOL(), changeUpkeepRewardPercent()`

* `StableConfig.sol`
Functions: `changeRewardPercentForCallingLiquidation(), changeMaxRewardValueForCallingLiquidation(), changeMinimumCollateralValueForBorrowing(), changeInitialCollateralRatioPercent(), changeMinimumCollateralRatioPercent(), changePercentArbitrageProfitsForStablePOL()`

* `PoolsConfig.sol`
Functions: `changeMaximumWhitelistedPools(), changeMaximumInternalSwapPercentTimes1000()`

* `RewardsConfig.sol`
Functions: `changeRewardsEmitterDailyPercent(), changeEmissionsWeeklyPercent(), changeStakingRewardsPercent(), changePercentRewardsSaltUSDS()`

* `StakingConfig.sol`
Functions: `changeMinUnstakeWeeks(), changeMaxUnstakeWeeks(), changeMinUnstakePercent(), changeModificationCooldown()`

The same issue has been identified also in `PriceAggregator.sol`:
Functions: `changeMaximumPriceFeedPercentDifferenceTimes1000()`, `changePriceFeedModificationCooldown()`.




# [17] Lack of data verification in `setPriceFeed()` in `PriceAggregator.sol`


[File: PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L53)
```
		if ( priceFeedNum == 1 )
			priceFeed1 = newPriceFeed;
		else if ( priceFeedNum == 2 )
			priceFeed2 = newPriceFeed;
		else if ( priceFeedNum == 3 )
			priceFeed3 = newPriceFeed;

		priceFeedModificationCooldownExpiration = block.timestamp + priceFeedModificationCooldown;
		emit PriceFeedSet(priceFeedNum, newPriceFeed);
```


Function `setPriceFeed()` does not validate `priceFeedNum` parameter. Since this function is called by only owner (`onlyOwner` modifier), the severity has been reduced to Low and included in QA report.

This parameter is responsible for setting a price feed (when it's 1 - function updates `priceFeed1`, when it's 2 - it updates `priceFeed2`, when it's 3 - it updates `priceFeed3`). However, when `priceFeedNum > 3` or `priceFeedNum == 0`, function won't update any price feed, but it will still emit `PriceFeedSet()` event. This behavior may be misleading to the end user.

Our recommendation is to revert when `priceFeedNum > 3` or `priceFeedNum == 0`: `require(priceFeedNum <= 3 && priceFeedNum > 0, "incorrect price feed");`



# [18] Standarize how `if` conditions are coded

There are two ways of coding `if`, when it contains a single instruction:

* with brackets: `if() { doSomething(); }`
* without brackets: `if() doSomething();`

The first way increases the code readability and is recommended.
This is extremely important when two `if` conditions are coded in a row, e.g.:

[File: StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L64)
```
		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
			[...]
			}
```

Having two `if` might be confusing to the code-reader. Our recommendation is to use `{}` and proper indentation in that case:

```
		if ( useCooldown )
		{
			if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
			[...]
			}
		}
```

The same issue occurs in lines 104-105:

```
		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
```




# [19] Incorrect grammar

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L239)
```
// The ballot is approved. Any reversions below will allow the ballot to be attemped to be finalized later - as the ballot won't be finalized on reversion.
```

`attemped` should be changed to `attempt`.

[File: Liquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L100)
```
// The added liquidity will be owned by this contract. (external call to Pools contract)
```

Start the new sentence (after the dot) with upper-cased letter:

```
// The added liquidity will be owned by this contract. (External call to Pools contract)
```


# [20] Use `&&` instead of two `if`-conditions in a row in `Proposals.sol`

[File: Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L372)
```
		if ( increaseTotal > decreaseTotal )
		if ( increaseTotal > noChangeTotal )
			return Vote.INCREASE;

		if ( decreaseTotal > increaseTotal )
		if ( decreaseTotal > noChangeTotal )
			return Vote.DECREASE;
```

Using two `if`-conditions in a row - without curly brackets (`{}`) decreases the code readability. It will be much better to use `&&` operator, since those conditions are basically the same:

```
		if ( increaseTotal > decreaseTotal && increaseTotal > noChangeTotal)
			return Vote.INCREASE;

		if ( decreaseTotal > increaseTotal && decreaseTotal > noChangeTotal)
			return Vote.DECREASE;
```



# [21] Incorrect punctuation in comments

[File: Liquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L105)
```
// Cooldown is specified to prevent reward hunting (ie - quickly [...]
```

should be changed to: `// Cooldown is specified to prevent reward hunting (ie., - quickly [...]`





# [22] It's possible to call `setInitialFeeds()` in `PriceAggregator.sol` multiple of times in some rare conditions

[File: PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L39)
```
		require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );

		priceFeed1 = _priceFeed1;
		priceFeed2 = _priceFeed2;
		priceFeed3 = _priceFeed3;
```

Function `setInitialFeeds()` should be called only once. This invariant, is however broken, when `_priceFeed1` is set to `address(0)`.

Function `setInitialFeeds()` does not check if parameter `_priceFeed1` is not `address(0)`. This means, that it's possible to call `setInitialFeeds()` multiple of times by passing `_priceFeed1` as `address(0)`.

When function `setInitialFeeds()` is called for the first time with `_priceFeed1` set to `address(0)` - `priceFeed1` will still point out to `address(0)`. This means, that it would be possible to call this function once again - even though, it should not be possible.

Please notice that it's possible to set `priceFeed2` and `priceFeed3` multiple of times, when `_priceFeed1` will always be set to `address(0)`.

E.g.

```
setInitialFeeds(address(0), address(0xAAA), address(0xBBB))
```

After the first call, `priceFeed2` and `priceFeed3` will be set to `0xAAA` and `0xBBB`. It should not be possible to call `setInitialFeeds()` again, but it's still possible:

```
setInitialFeeds(address(0), address(0x123), address(0x456))
```

After the 2nd call, `priceFeed2` and `priceFeed3` will be set to `0x123`, `0x456`.

Due to very rare likelihood (only owner needs to call `setInitialFeeds()` with `_priceFeed1` set to `address(0)`) - this issue has been reported in QA reports only as bad coding practice.

Nonetheless, our recommendation is to add additional check which requires that `_priceFeed1` cannot be `address(0)`: `require(_priceFeed1 != address(0), "price feed cannot be addr(0)")`.


# [23] Off-by-one error when checking if quorum is reached

Function `requiredQuorumForBallotType()` defines how much votes we need to reach the quorum. It's being used in `winningParameterVote()` to verify if quorum is reached:

[File: Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L396)
```
        if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
            return false;
```

Reaching the quorum - by definition - means that the proposal has at least the same or more votes than the quorum defines.
However, using `<` instead of `<=` operator means that proposal needs to exceed the defined quorum to be accepted.

E.g. if quorum is defined as `100` (function `requiredQuorumForBallotType()` returns 100), then when `totalVotesCastForBallot(ballotID)` returns `100`, the quorum should be reached (we had defined quorum to 100, thus when there are 100 votes, quorum is reached).
However, in `Proposals.sol` contract, (because of the `<` operator), function `totalVotesCastForBallot(ballotID)` needs to return `101` - to make the quorum be reached. This is an off-by-one issue related to the quorum definition.
Even though it does not have any security related implications - our recommendation is to change `<` to `<=` operator.



# [24] `winningParameterVote()` from `Proposals.sol` may return inconsistent result

[File: Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L372)
```
if ( increaseTotal > decreaseTotal )
		if ( increaseTotal > noChangeTotal )
			return Vote.INCREASE;

		if ( decreaseTotal > increaseTotal )
		if ( decreaseTotal > noChangeTotal )
			return Vote.DECREASE;

		return Vote.NO_CHANGE;
```

Above conditions does not correctly choose the winning parameter vote when there's a tie. Let's consider 4 possible scenarios:

```
votes[Vote.INCREASE]  = 10000
votes[Vote.DECREASE]  = 10000
votes[Vote.NO_CHANGE] = 10000
```

Since `Vote.INCREASE == Vote.DECREASE == Vote.NO_CHANGE` function will return `Vote.NO_CHANGE`. This is reasonable, since there are equally the same votes for each options - it's ambiguous what do to - thus no change.

```
votes[Vote.INCREASE]  = 1
votes[Vote.DECREASE]  = 10000
votes[Vote.NO_CHANGE] = 10000
```

Since `Vote.INCREASE == Vote.Vote.NO_CHANGE` and there's is very small amount of votes for `Vote.INCREASE`, function will return `Vote.NO_CHANGE`. This is reasonable. Since almost no one voted for `Vote.INCREASE` and there's a tie between `Vote.DECREASE` and `Vote.NO_CHANGE` - thus no change.

```
votes[Vote.INCREASE]  = 10000
votes[Vote.DECREASE]  = 1
votes[Vote.NO_CHANGE] = 10000
```

The same reasoning as above.

```
votes[Vote.INCREASE]  = 10000
votes[Vote.DECREASE]  = 10000
votes[Vote.NO_CHANGE] = 1
```

Here is, however, where the result is incorrect. Let's check the exact flow:

`if ( increaseTotal > decreaseTotal ) if ( increaseTotal > noChangeTotal )` is `false`.
`if ( decreaseTotal > increaseTotal ) if ( decreaseTotal > noChangeTotal )`  is `false`.
Thus function returns `Vote.NO_CHANGE`. This is, however, incorrect. Please notice, that multiple of people voted on either `Vote.INCREASE` or `Vote.DECREASE` and almost no one on `Vote.NO_CHANGE` - but still, the winning is `Vote.NO_CHANGE`. Consider having additional rule which will prioritize either `Vote.INCREASE` or `Vote.DECREASE` in case of tie, when `Vote.DECREASE == Vote.INCREASE` and `Vote.NO_CHANGE` is very small, function should return either `Vote.DECREASE` or `Vote.INCREASE`.



# [25] The same pool can be whitelisted multiple of times in `PoolsConfig.sol`

Function `whitelistPool()` utilizes OZ's EnumerableSet implementation. If the set already contains the value which protocol tries to add - function `add()` will return `false` (instead of reverting). This implies that function `whitelistPool()` from `PoolsConfig.sol` can be called multiple of times, with the same pool.
Basically, it won't pose any security risk (since the `pool` would had already been added to `_whitelist`) - thus this issue is only reported in the QA report.

A good security practice is to verify if the value is already in set and revert if it is. This could be achieved by changing below code:

[File: PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L53)
```
_whitelist.add(poolID);
```

to:

```
require(_whitelist.add(poolID), "already added");
```


# [26] It's possible to unwhitelist non-whitelisted pool

Function `unwhitelistPool()` utilizes OZ's EnumerableSet implementation. If the set does not contain the value which protocol tries to remove - function `remove()` will return `false` (instead of reverting). This implies that function `unwhitelistPool()` from `PoolsConfig.sol` can be called multiple of times, with the same pool or can be called on the pool which hadn't been whitelisted. During the every call (even for non-whitelisted pools), function will emit `PoolUnwhitelisted()` event. This behavior may be very misleading to the end-user.
If pool does not exist and/or is not whitelisted, calling `unWhitelistPool()` should revert, instead of emitting `PoolUnwhitelisted()` event

A good security practice is to verify if the value is already in set and revert if we're trying to remove the element which does not exist. This could be achieved by changing below code:

[File: PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L67)
```
_whitelist.remove(poolID);
```

to:

```
require(_whitelist.remove(poolID), "pool not whitelisted");
```


# [27] Unclear comment in `PoolMath.sol`

[File: PoolMath.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolMath.sol#L186)
```
		// Here for reference
//        uint256 C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
//        uint256 discriminant = B * B - 4 * A * C;

		// Negate C (from above) and add instead of subtract.
		// r1 * z0 guaranteed to be greater than r0 * z1 per the conditional check in _determineZapSwapAmount
        uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
        uint256 discriminant = B * B + 4 * A * C;
```

Comment states that: `//        uint256 discriminant = B * B - 4 * A * C;` (line 188)
However, the code at line 193 is defined as below: `uint256 discriminant = B * B + 4 * A * C;`.
As demonstrated, we're adding, instead of subtracting `4 * A * C`. However, when we take a deeper look at another comment, at line 190: `// Negate C (from above) and add instead of subtract.`, we can see, that indeed, we need to add, instead of subtract. 
The structure of the comment is, however, extremely misleading. When some piece of comment references to the exact math calculation, it should fully explain the code in the same comment.
Do not use two different comments to describe the same calculation, especially, when those comments are contradicting themselves.
The much better idea would be to comment above math as a single comment:

```
		// Here for reference 
//        uint256 C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
//        uint256 discriminant = B * B + 4 * A * C; /// C is negated, thus add instead of subtract
```


# [28] Redundant calculations in `RewardsEmitter.sol`

[File: RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L115)
```
115: 	uint256 sum = 0;
116:	for( uint256 i = 0; i < poolIDs.length; i++ )
[...]
128: 			sum += amountToAddForPool;
```

Function `performUpkeep()` - at line 115 - initializes variable `sum` and then, in every loop iteration - it increases its value by `amountToAddForPool` - line 128.
However, the further code analysis demonstrated, that this variable is nowhere used. Initialization of this variable (line 115) and adding `amountToAddForPool` to it (line 128) is redundant, because `performUpkeep()` does not utilize this variable at all. 
Removing an unused variables and unused functionality of the code (keeping the sum of `amountToAddForPool` in `sum` variable) increases the code readability. Our recommendation is to remove line 115 and 128 from the `performUpkeep()`.





# [29] Typos 

[File: SigningTools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/SigningTools.sol#L10C51-L10C65)
```
// Verify that the messageHash was signed by the authoratative signer.
```

`authoratative` should be changed to `authoritative`.


# [30] Amount required to confirm wallet is hardcoded in the contract, thus it's not possible to change it in the future

[File: ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L65)
```
    	if ( msg.value >= .05 ether )
    		activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
```

Value is hardcoded to `0.05` ether. If ETH value increases in its value in the future, `0.05` might be too expensive value to pay to confirm a wallet. Our recommendation is to implement a way to update this value in the future and do not hard-code it directly into the code.





# [31] Standardize syntax for `for`-loops

Good coding practice requires to use white-space between `for` keyword and parenthesis. This is done only in `CollateralAndLiquidity.sol`
```
./src/stable/CollateralAndLiquidity.sol:321:                    for ( uint256 i = startIndex; i <= endIndex; i++ )
./src/stable/CollateralAndLiquidity.sol:338:            for ( uint256 i = 0; i < count; i++ )
```

The rest of the code-base uses `for(` instead of `for (`.

```
./src/arbitrage/ArbitrageSearch.sol:115:                        for( uint256 i = 0; i < 8; i++ )
./src/dao/Proposals.sol:423:            for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
./src/pools/PoolStats.sol:53:           for( uint256 i = 0; i < poolIDs.length; i++ )
./src/pools/PoolStats.sol:65:           for( uint256 i = 0; i < poolIDs.length; i++ )
./src/pools/PoolStats.sol:81:           for( uint256 i = 0; i < poolIDs.length; i++ )
./src/pools/PoolStats.sol:106:          for( uint256 i = 0; i < poolIDs.length; i++ )
./src/rewards/RewardsEmitter.sol:60:            for( uint256 i = 0; i < addedRewards.length; i++ )
./src/rewards/RewardsEmitter.sol:116:           for( uint256 i = 0; i < poolIDs.length; i++ )
./src/rewards/RewardsEmitter.sol:146:           for( uint256 i = 0; i < rewards.length; i++ )
./src/rewards/SaltRewards.sol:64:               for( uint256 i = 0; i < addedRewards.length; i++ )
./src/rewards/SaltRewards.sol:87:               for( uint256 i = 0; i < addedRewards.length; i++ )
./src/rewards/SaltRewards.sol:129:              for( uint256 i = 0; i < poolIDs.length; i++ )
./src/staking/Staking.sol:163:        for(uint256 i = start; i <= end; i++)
./src/staking/StakingRewards.sol:152:           for( uint256 i = 0; i < poolIDs.length; i++ )
./src/staking/StakingRewards.sol:185:           for( uint256 i = 0; i < addedRewards.length; i++ )
./src/staking/StakingRewards.sol:216:           for( uint256 i = 0; i < shares.length; i++ )
./src/staking/StakingRewards.sol:226:           for( uint256 i = 0; i < rewards.length; i++ )
./src/staking/StakingRewards.sol:260:           for( uint256 i = 0; i < rewards.length; i++ )
./src/staking/StakingRewards.sol:277:           for( uint256 i = 0; i < shares.length; i++ )
./src/staking/StakingRewards.sol:296:           for( uint256 i = 0; i < cooldowns.length; i++ )
```

# [32] Use `uint256` instead of `uint`
While `uint` is a shorter form of `uint256`, using the latter increases the code readability:

Almost the whole implementation uses `uint256`. The shorter form was noticed only in two places:
```
./src/pools/Pools.sol:81:       modifier ensureNotExpired(uint deadline)
./src/staking/Liquidity.sol:40: modifier ensureNotExpired(uint deadline)
```