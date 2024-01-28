# [G-01] Claiming airdrops in `Airdrop.sol` can be fully rewritten and optimized

`Airdrop.sol` utilizes mapping `claimed` to check if user has already claimed some token:

[File: Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L29)
```
	// Those users who have already claimed
	mapping(address=>bool) public claimed;
```

Now, in function `claimAirdrop()`, we're checking this mapping (and update it):

[File: Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L74)
```
function claimAirdrop() external nonReentrant
    	{
    	require( claimingAllowed, "Claiming is not allowed yet" );
    	require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
    	require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );

		// Have the Airdrop contract stake a specified amount of SALT and then transfer it to the user
		staking.stakeSALT( saltAmountForEachUser );
		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );

    	claimed[msg.sender] = true;
    	}
```

This is, however - redundant, since we already have a list of users who can claim airdrop in `_authorizedUsers`.
When claiming is allowed, no other users can be added to `_authorizedUsers` (function `authorizeWallet()` does not allow to add new wallets when claiming is allowed).
This constitutes a very simple and elegant solution. Whenever we call `claimAirdrop()` - we can just remove user from `_authorizedUsers`.

Please notice, that we already check if user is on `_authorizedUsers`: (line 77) `require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );`.
If after claiming an airdrop, we would remove it from `_authorizedUsers`, he won't be able to call `claimAirdrop()` again.
Moreover, since `_authorizedUsers` is Open Zeppelin's enumerable set, removing element from that set costs O(1).

To sum up, function `claimAirdrop()` can be rewritten:

```
function claimAirdrop() external nonReentrant
    	{
    	require( claimingAllowed, "Claiming is not allowed yet" );
    	require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
    	

		// Have the Airdrop contract stake a specified amount of SALT and then transfer it to the user
		staking.stakeSALT( saltAmountForEachUser );
		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );

    	_authorizedUsers.remove(msg.sender);
    	}
```

# [G-02] `updateArbitrageIndicies()` from `PoolStats.sol` can be fully optimized

[File: PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L89)
```
				uint64 poolIndex1 = _poolIndex( _weth, arbToken2, poolIDs );
				uint64 poolIndex2 = _poolIndex( arbToken2, arbToken3, poolIDs );
				uint64 poolIndex3 = _poolIndex( arbToken3, _weth, poolIDs );

				// Check if the indicies in storage have the correct values - and if not then update them
				ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
				if ( ( poolIndex1 != indicies.index1 ) || ( poolIndex2 != indicies.index2 ) || ( poolIndex3 != indicies.index3 ) )
```

Solidity uses short-circuit to execute statements in conditions. This means, that for expression: `A or B`, when `A` evaluates to `true` - evaluation of `B` will be omitted (because `true OR whatever` is always `true`, thus there's no need to evaluate `B`).

This behavior means, that we don't need to calculate every `_poolIndex()` at once, but use this directly in the `if` statement. This will save us gas because: firstly, we won't be declaring redundant variables `poolIndex1`, `poolIndex2`, `poolIndex3` - which are used only once - and secondly - because of the short-circuiting, when first expression returns `true`, we won't be performing another 2 `_poolIndex()` calls:

```
if ( ( _poolIndex( _weth, arbToken2, poolIDs ) != indicies.index1 ) || ( _poolIndex( arbToken2, arbToken3, poolIDs ) != indicies.index2 ) || ( _poolIndex( arbToken3, _weth, poolIDs ) != indicies.index3 ) )
```

# [G-03] Use binary search in `_executeParameterChange()` in `Parameter.sol`

Since `ParameterTypes` is `enum`, its value represents consecutive integers. Instead of performing 26 comparisons (there are 1 `if` and 25 `else-if` conditional checks in `_executeParameterChange()`) - it might be a better idea to implement a binary-search in O(log n). The current time complexity is linear - O(n).

# [G-04] `Proposals.sol` should be fully redesign by separating its functionality

The current implementation of `Proposals.sol` looks as follows. Every proposal: `createConfirmationProposal()`, `proposeParameterBallot()`, `proposeTokenWhitelisting()`, `proposeTokenUnwhitelisting()`, `proposeSendSALT()`, `proposeCallContract()`, `proposeCountryInclusion()`, `proposeCountryExclusion()`, `proposeSetContractAddress()`, `proposeWebsiteUpdate()` calls internal function `_possiblyCreateProposal()`. This function is like a prototype for creating proposal - it contains every parameter for every proposal type.

[File: Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81)
```
function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 )
```

however, proposals do not need all these parameters.

E.g. when we call `proposeTokenWhitelisting()`, we don't care about `uint256 number1` - but we still need to pass it as `0`: `_possiblyCreateProposal( ballotName, BallotType.WHITELIST_TOKEN, address(token), 0, tokenIconURL, description );`.

When we call `proposeCountryInclusion()`, we don't care about `address address1` and `uint256 number1` - but we still need to pass it as `address(0)` and `0`: `_possiblyCreateProposal( ballotName, BallotType.EXCLUDE_COUNTRY, address(0), 0, country, description );`.

This occurs for every proposal. In many proposals, we waste gas on passing redundant parameters (like `0`, `address(0)`) to `_possiblyCreateProposal()`, which are not needed for the type of proposal we want.

Much better idea would be to save gas by coding each proposal directly. Even though we would spend more gas on the deployment (the code would be bigger) - we would save gas when executing contract (it wouldn't be needed to pass redundant parameters to internal function - actually, we won't need to call `_possiblyCreateProposal()` at all).

Our recommendation is to remove `_possiblyCreateProposal()` and implement each proposal creation directly in functions `createConfirmationProposal()`, `proposeParameterBallot()`, `proposeTokenWhitelisting()`, `proposeTokenUnwhitelisting()`, `proposeSendSALT()`, `proposeCallContract()`, `proposeCountryInclusion()`, `proposeCountryExclusion()`, `proposeSetContractAddress()`, `proposeWebsiteUpdate()`; instead of calling `_possiblyCreateProposal()` there.



# [G-05] `tokenWhitelistingBallotWithTheMostVotes()` in `Proposals.sol` can be optimized

[File: Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L429)
```
			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
```

In the list of ballots, it's very likely that there would be a lot of ballots which have more `yes`es than `no`s. Also, it's very likely that there would be a lot of ballots which reached the quorum. However, there will be only one ballot with the most `yes` votes. This implies, that we firstly should check `yesTotal > mostYes`.

If the current ballot has less `yesTotal` than `mostYes`, we do not care if it reached quorum or if the vote is favorable. Even if it is - it still not be the ballot with most `yes`es.
This implies, that `yesTotal > mostYes` should be checked first. 

Secondly, we don't care if quorum is reached, when there's more `noTotal` than `yesTotal`, thus `yesTotal > noTotal` should be evaluated before `yesTotal + noTotal >= quorum`.

To sum up, above code should be changed to:

```
if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
if ( yesTotal > noTotal )  // Make sure the token vote is favorable
if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
```


# [G-06] Every `change*` function from `*Config.sol` should be fully optimized


Every file from the below list:
```
./src/dao/DAOConfig.sol
./src/ExchangeConfig.sol
./src/pools/PoolsConfig.sol
./src/rewards/RewardsConfig.sol
./src/stable/StableConfig.sol
./src/staking/StakingConfig.sol
```

contains a public state variable which defines some protocol configuration. E.g., let's take a look at `RewardsConfig.sol`. It defines multiple of configurations:

```
uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;
uint256 public emissionsWeeklyPercentTimes1000 = 500;
uint256 public stakingRewardsPercent = 50;
uint256 public percentRewardsSaltUSDS = 10;
```

Each of these configuration can be updated by calling a proper `change*` function. E.g., to update `rewardsEmitterDailyPercentTimes1000` - we call `changeRewardsEmitterDailyPercent()`, to update 
`emissionsWeeklyPercentTimes1000`, we call `changeEmissionsWeeklyPercent()` and so on.

Each of these functions is implemented basically in the same way:

[File: RewardsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsConfig.sol#L52)
```
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
```

It takes `bool increase` parameter which defines if we'll increase or decrease the state variable.
Then it increase (of decrease) with previously defined adjustment. In the above example, we can increase/decrease `emissionsWeeklyPercentTimes1000` by 250.
Before each increase/decrease - function checks if the state variable will still be within previously defined range. E.g., for  `emissionsWeeklyPercentTimes1000` the maximum and minimum range is 1000 and 250.
Let's examine some example to understand that:
When we want to decrease `emissionsWeeklyPercentTimes1000`, it has to be greater than 250 (otherwise, decreasing it will make it not be within range). The same - when we want to increase it.


However, please notice that function reads the state variable multiple of times, instead of caching it and store in the local variable.
When we want to increase/decrease `emissionsWeeklyPercentTimes1000` variable - firstly we read its value in `if` statement. Secondly, we read its value to either increase/decrease it, then we read that value to assign the new value to it and finally, we read `emissionsWeeklyPercentTimes1000` to emit `EmissionsWeeklyPercentChanged()` event.

Everything will use much less gas, when we will be able to cache the variable and move the event into `if` condition:

```
	function changeEmissionsWeeklyPercent(bool increase) external onlyOwner
        {
        uint256 emissionsWeeklyPercentTimes1000Cached = emissionsWeeklyPercentTimes1000;
        if (increase)
            {
            if (emissionsWeeklyPercentTimes1000Cached < 1000)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000Cached + 250;
                emit EmissionsWeeklyPercentChanged(emissionsWeeklyPercentTimes1000Cached + 250);
            }
        else
            {
            if (emissionsWeeklyPercentTimes1000Cached > 250)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000Cached - 250;
                emit EmissionsWeeklyPercentChanged(emissionsWeeklyPercentTimes1000Cached - 250);
            }

		
        }
```

This issue affects basically every `change*` function in every `*Config.sol` file. Every `change*` function is implemented like below:

```
	function changeSomething(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (stateVariable < RANGE_MAX)
                stateVariable = stateVariable + AMOUNT;
            }
        else
            {
            if (stateVariable > RANGE_MIN)
                stateVariable = stateVariable - AMOUNT;
            }

		emit Event(stateVariable);
        }
```

and can be rewritten to form:

```
	function changeSomething(bool increase) external onlyOwner
        {
        uint256 localVariableCache = stateVariable;
        if (increase)
            {
            if (localVariableCache < RANGE_MAX)
                stateVariable = localVariableCache + AMOUNT;
                emit Event(localVariableCache + AMOUNT);
            }
        else
            {
            if (localVariableCache > RANGE_MIN)
                stateVariable = localVariableCache - AMOUNT;
                emit Event(localVariableCache - AMOUNT);
            }

		emit Event(stateVariable);
        }
```

List of functions which should be optimized:

```
./src/dao/DAOConfig.sol:64:     function changeBootstrappingRewards(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:81:     function changePercentPolRewardsBurned(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:98:     function changeBaseBallotQuorumPercent(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:115:    function changeBallotDuration(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:132:    function changeRequiredProposalPercentStake(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:149:    function changeMaxPendingTokensForWhitelisting(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:166:    function changeArbitrageProfitsPercentPOL(bool increase) external onlyOwner
./src/dao/DAOConfig.sol:183:    function changeUpkeepRewardPercent(bool increase) external onlyOwner
./src/pools/PoolsConfig.sol:77: function changeMaximumWhitelistedPools(bool increase) external onlyOwner
./src/pools/PoolsConfig.sol:94: function changeMaximumInternalSwapPercentTimes1000(bool increase) external onlyOwner
./src/rewards/RewardsConfig.sol:36:     function changeRewardsEmitterDailyPercent(bool increase) external onlyOwner
./src/rewards/RewardsConfig.sol:52:     function changeEmissionsWeeklyPercent(bool increase) external onlyOwner
./src/rewards/RewardsConfig.sol:69:     function changeStakingRewardsPercent(bool increase) external onlyOwner
./src/rewards/RewardsConfig.sol:86:     function changePercentRewardsSaltUSDS(bool increase) external onlyOwner
./src/stable/StableConfig.sol:47:       function changeRewardPercentForCallingLiquidation(bool increase) external onlyOwner
./src/stable/StableConfig.sol:67:       function changeMaxRewardValueForCallingLiquidation(bool increase) external onlyOwner
./src/stable/StableConfig.sol:84:       function changeMinimumCollateralValueForBorrowing(bool increase) external onlyOwner
./src/stable/StableConfig.sol:101:      function changeInitialCollateralRatioPercent(bool increase) external onlyOwner
./src/stable/StableConfig.sol:118:      function changeMinimumCollateralRatioPercent(bool increase) external onlyOwner
./src/stable/StableConfig.sol:138:      function changePercentArbitrageProfitsForStablePOL(bool increase) external onlyOwner
./src/staking/StakingConfig.sol:34:     function changeMinUnstakeWeeks(bool increase) external onlyOwner
./src/staking/StakingConfig.sol:51:     function changeMaxUnstakeWeeks(bool increase) external onlyOwner
./src/staking/StakingConfig.sol:68:     function changeMinUnstakePercent(bool increase) external onlyOwner
./src/staking/StakingConfig.sol:85:     function changeModificationCooldown(bool increase) external onlyOwner
```

# [G-07] Redundant calculations in `RewardsEmitter.sol`

[File: RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L115)
```
115: 	uint256 sum = 0;
116:	for( uint256 i = 0; i < poolIDs.length; i++ )
[...]
128: 			sum += amountToAddForPool;
```

Function `performUpkeep()` intiializes variable `sum` and then, in every loop iteration - it adds the `amountToAddForPool` value to it.
However, it seems that variable `sum` is nowhere used. Declaration of this variable (line 115) and adding `amountToAddForPool` to it (line 128) is redundant and can be removed.
Function `performUpkeep()` does not utilize this variable at all, thus it can be simply removed.




# [G-08] `excludedCountries` should be a different type

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L67)
```
mapping(string=>bool) public excludedCountries;
```

Since `excludedCountries` contains ISO 3166 Alpha-2 codes, which are 2-letters codes, the key type of this mapping should be `bytes32` (or `byte2`).
Moreover, it's more profitable in case of gas to operate on `uint256` than `bool`, thus the value type of this mapping should be `uint256`

# [G-09] Use binary search in `_executeApproval()` in `DAO.sol`

Since `BallotType` is `enum`, its value represents consecutive integers. Instead of performing 9 comparisons (there are 1 `if` and 8 `else-if` conditional checks in `_executeApproval()`) - it might be a better idea to implement a binary-search in O(log n). The current time complexity is linear - O(n).


# [G-10] `require` statements which uses less gas should be called first

[File: CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L117-L119)
```
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		require( amountRepaid > 0, "Cannot repay zero amount" );
```

Reading state variables (e.g. `usdsBorrowedByUsers[msg.sender]`) uses more gas than reading function parameter (e.g. `amountRepaid`). This implies, that `amountRepaid > 0` costs less than `amountRepaid <= usdsBorrowedByUsers[msg.sender]`. Calling `userShareForPool()` costs even more. Above code should be rewritten to:

```
		require( amountRepaid > 0, "Cannot repay zero amount" );
		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
```

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L142)
```
		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( exchangeIsLive, "The exchange is not yet live" );
		require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );
```

Reading state variable `collateralAndLiquidity` and `exchangeIsLive` uses more gas than reading function's parameters `tokenA`, `tokenB`. This implies that order of `require` statements should be changed:

```
require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );
require( exchangeIsLive, "The exchange is not yet live" );
require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
```

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L172)
```
		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
```

Reading state variable `collateralAndLiquidity` uses more gas than reading function's parameter `liquidityToRemove`, thus the order of `require` statements should be changed.

[File: StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L59)
```
		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
		require( increaseShareAmount != 0, "Cannot increase zero share" );
```

Calling function `isWhitelisted()` uses more gas than reading function's parameter, thus the order of `require` statements should be changed.



# [G-11] Change `>` to `>=` to avoid redundant `return`

[File: StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L248)
```
		if ( user.virtualRewards > rewardsShare )
			return 0;

		return rewardsShare - user.virtualRewards;
```

When `user.virtualRewards == rewardsShare`, then the 2nd `return` will also return `0`. Changing above code to:

```
		if ( user.virtualRewards >= rewardsShare )
			return 0;

		return rewardsShare - user.virtualRewards;
```

will save us a redundant substraction, when `user.virtualRewards == rewardsShare` (in that context, function will return with `0` ealier).


# [G-12] It's possible to save one reading of `ballotFinalized` in `BootstrapBallot.sol` by changing its type

[File: BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L69)
```
bool public ballotFinalized;
[...]
require( ! ballotFinalized, "Ballot has already been finalized" );
[...]
ballotFinalized = true;
```

`ballotFinalized` is used only in `finalizeBallot()` function. It's a state variable, thus reading it might be costly. If we change its type to `uint256`, we can save one reading by using `++` operator directly in `require` statement:

```
uint256 public ballotFinalized = 1;  // @audit: 1 means not finalized, 2 means finalized, we use 1;2 do avoid changing state from 0 to 1 in finalizeBallot()
[...]
require(ballotFinalized++ == 1, "Ballot has already been finalized" ); // @audit: on the second call, ballotFinalized will be 2, thus function will revert.
[...]
// @audit: this is no needed anymore: ballotFinalized = true;
```

# [G-13] `step6()` in `Upkeep.sol` can be optimized

[File: Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L186)
```
	function step6() public onlySameContract
		{
		uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions;
		emissions.performUpkeep(timeSinceLastUpkeep);

		lastUpkeepTimeEmissions = block.timestamp;
		}
```

There are two optimizations available.
* Since `lastUpkeepTimeEmissions` is set to `block.timestamp` in the `constructor()` and later in `step6()`, we can be sure that `lastUpkeepTimeEmissions <= block.timestamp`, thus `block.timestamp - lastUpkeepTimeEmissions` can be unchecked.

* There's no need to declare local `timeSinceLastUpkeep` variable, since it's used only once

```
	function step6() public onlySameContract
		{
		unchecked {
			emissions.performUpkeep(block.timestamp - lastUpkeepTimeEmissions);
			}
		lastUpkeepTimeEmissions = block.timestamp;
		}
```


# [G-14] Use tautology to save one negation

Due to de Morgans Law: `not A AND not B <=> not (A OR B)`. This means that below line of code:

[File: PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L136)
```
require(address(pair.tokenA) != address(0) && address(pair.tokenB) != address(0), "This poolID does not exist");
```

can be changed to:

```
require(  !(address(pair.tokenA) == address(0) || address(pair.tokenB) == address(0)), "This poolID does not exist");
```

to save one negation.


[File: PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L87)
```
if ( (arbToken2 != _weth) && (arbToken3 != _weth) )
```

`(arbToken2 != _weth) && (arbToken3 != _weth)` is the same as `! ( (arbToken2 == _weth) || (arbToken3 == _weth) )`


# [G-15] Redundant safety check in `PoolMath.sol`

[File: PoolMath.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolMath.sol#L193)
```
 uint256 discriminant = B * B + 4 * A * C;

        // Compute the square root of the discriminant.
        uint256 sqrtDiscriminant = Math.sqrt(discriminant);

		// Safety check: make sure B is not greater than sqrtDiscriminant
		if ( B > sqrtDiscriminant )
			return 0;
```

Both `A`, `B`, `C` are declared as `uint56`, thus these variables are `>= 0`.
We know that square root of `B * B` is equal to `B`. Thus square root of `B * B + whatever` has to be greater or equal (in case of rounding down) than `B`.
This case is fulfilled here. `Math.sqrt(B * B + 4 * A * C) >= B`. This means that `if ( B > sqrtDiscriminant )` will only be fulfilled when `B == sqrtDiscriminant`.
Now let's check what will happen when `B == sqrtDiscriminant`. In line 203: `swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );` we will have `swapAmount = 0 / ( 2 * A ) = 0`.
This implies that function will return `0`, the same as at line 199. This means, that lines 199-200 are redundant.

```
		// Safety check: make sure B is not greater than sqrtDiscriminant
		if ( B > sqrtDiscriminant )
			return 0;
```

Above safety check can be removed.




# [G-16] Do not declare variables inside the loop

Variables should be declared outside the loop, so that they won't be re-declared on every loop iteration.

[File: ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L115)
```
			for( uint256 i = 0; i < 8; i++ )
				{
				uint256 midpoint = (leftPoint + rightPoint) >> 1;
```

should be changed to:

```
			uint256 midpoint;
			for( uint256 i = 0; i < 8; i++ )
				{
				midpoint = (leftPoint + rightPoint) >> 1;
```


[File: Proposal.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L423)
```
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];
```

should be changed to:

```
			uint256 ballotID;
			uint256 yesTotal;
			uint256 noTotal;
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			ballotID = _openBallotsForTokenWhitelisting.at(i);
			yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			noTotal = _votesCastForBallot[ballotID][Vote.NO];
```


The same issue occurs in:
* `PoolStats.sol` - functions `updateArbitrageIndicies()` and `_calculateArbitrageProfits()`
* `RewardsEmitter.sol` - functions `addSALTRewards()` and `performUpkeep()`
* `SaltRewards.sol` - function `_sendLiquidityRewards()` 
* `StakingRewards.sol` - functions `claimAllRewards()`, `addSALTRewards()` and `userCooldowns()`
* `CollateralAndLiquidity.sol` function `findLiquidatableUsers()`





# [G-17] Increasing counters can be unchecked

[File: AccessManager.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/AccessManager.sol#L45)
```
 geoVersion += 1;
```

Since `geoVersion` is being changed only at every DAO updates, it's almost impossible that this value would ever overflow - thus it can be unchecked.


[File: BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L56)
```

		if ( voteStartExchangeYes )
			startExchangeYes++;
		else
			startExchangeNo++;
```

There's no enough voters to these value ever overflow - thus they can be unchecked.


[File: PriceAggeregator.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L113)
```
		if (price1 > 0)
			numNonZero++;

		if (price2 > 0)
			numNonZero++;

		if (price3 > 0)
			numNonZero++;
```

`numNonZero++` can be unchecked.


[File: Staking.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L67)
```
unstakeID = nextUnstakeID++;
```

This won't overflow, thus can be unchecked.

# [G-18] There's no need to declare variables used only once

[File: AccessManager.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/AccessManager.sol#L53)
```
		bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));

		return SigningTools._verifySignature(messageHash, signature);
```

`messageHash` is used only once, thus there's no need to declare it at all:

```
return SigningTools._verifySignature(keccak256(abi.encodePacked(block.chainid, geoVersion, wallet)), signature);
```

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L246)
```
			uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

			// Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

[...]
			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );

			// Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
			AddedReward[] memory addedRewards = new AddedReward[](2);
			addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
			addedRewards[1] = AddedReward( pool2, bootstrappingRewards );
```

Variables `saltBalance`, `bestWhitelistingBallotID`, `pool1`, `pool2` are used only once, thus there's no need to declare them at all:

```
			require( exchangeConfig.salt().balanceOf( address(this) ) >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

			// Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
			require( proposals.tokenWhitelistingBallotWithTheMostVotes() == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

[...]

			// Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
			AddedReward[] memory addedRewards = new AddedReward[](2);
			addedRewards[0] = AddedReward( PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() ), bootstrappingRewards );
			addedRewards[1] = AddedReward( PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() ), bootstrappingRewards );
```


[File: PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L146)
```
		bytes32 poolID1 = PoolUtils._poolID( token, wbtc );
		if ( isWhitelisted(poolID1) )
			return true;

		bytes32 poolID2 = PoolUtils._poolID( token, weth );
		if ( isWhitelisted(poolID2) )
```

Variables `poolID1` and `poolID2` are used only once, thus there no need to declare them at all:

```
		if ( isWhitelisted(PoolUtils._poolID( token, wbtc )) )
			return true;

		if ( isWhitelisted(PoolUtils._poolID( token, weth )) )
```


# [G-19] `require` statemets which uses less gas should be executed first

Whenever we use 2 `require` statements, we assume that both of them won't fail. If the first one will fail (thus function will revert), the second one won't be executed. Based on that, it's better to execute the `require` statement which uses less gas first. If it would revert, user wouldn't be paying gas for the 2nd (more expensive) revert.

[File: ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L44)
```
		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
```

Reading state variable costs more than reading a function parameter. This means, that above code should be changed to:

```
		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
```


The same issue occurs in other instances:

[File: USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L42)
```
		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
		require( amount > 0, "Cannot mint zero USDS" );
```

Reading stare variable `address(collateralAndLiquidity)` costs more than function's parameter `amount`, thus order of the `require` statements should be changed.




# [G-20] Calculations which won't underflow should be unchecked

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L341)
```
		uint256 amountToSendToTeam = claimedSALT / 10;
		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), amountToSendToTeam );
		emit TeamRewardsTransferred(amountToSendToTeam);

		uint256 remainingSALT = claimedSALT - amountToSendToTeam;
```

Since `amountToSendToTeam = claimedSALT / 10`, we know that `claimedSALT >= amountToSendToTeam`, thus line `uint256 remainingSALT = claimedSALT - amountToSendToTeam` can be unchecked.


[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L221)
```
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;
```

Because `_userDeposits[msg.sender][token] >= amount`, `_userDeposits[msg.sender][token] -= amount` won't underflow and can be unchecked.

[File: SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L143)
```
uint256 stakingRewardsAmount = ( remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100;
uint256 liquidityRewardsAmount = remainingRewards - stakingRewardsAmount;
```

`rewardsConfig.stakingRewardsPercent()` is defined as value in range `[25, 75]`. This means that `remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100` will always be lower or equal to `remainingRewards`. According to that `remainingRewards - stakingRewardsAmount` won't underflow, thus can be unchecked.



[File: CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L118)
```
		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		require( amountRepaid > 0, "Cannot repay zero amount" );

		// Decrease the borrowed amount for the user
		usdsBorrowedByUsers[msg.sender] -= amountRepaid;
```

Because `amountRepaid <= usdsBorrowedByUsers[msg.sender]`, `usdsBorrowedByUsers[msg.sender] -= amountRepaid` won't underflow and can be unchecked.


[File: StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L300)
```
if ( block.timestamp >= cooldownExpiration )
				cooldowns[i] = 0;
			else
				cooldowns[i] = cooldownExpiration - block.timestamp;
			}
```

Because `block.timestamp < cooldownExpiration`, `cooldownExpiration - block.timestamp` won't underflow and can be unchecked.

# [G-21] Do not perform `safeTransfer` when amount is 0

0-transfers does not change anything and are considered as a waste of gas. Before performing a transfer - make sure that the transferred amount is greater than 0.

[File: DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L372)
```
		(uint256 reclaimedA, uint256 reclaimedB) = collateralAndLiquidity.withdrawLiquidityAndClaim(tokenA, tokenB, liquidityToWithdraw, 0, 0, block.timestamp );

		// Send the withdrawn tokens to the Liquidizer so that the tokens can be swapped to USDS and burned as needed.
		tokenA.safeTransfer( address(liquidizer), reclaimedA );
		tokenB.safeTransfer( address(liquidizer), reclaimedB );
```

Add additional check:

```
if (reclaimedA > 0) tokenA.safeTransfer( address(liquidizer), reclaimedA );;
if (reclaimedB > 0) tokenB.safeTransfer( address(liquidizer), reclaimedB );
```

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L161)
```
		tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
		tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );
```

Add additional check which guarantees that `addedAmountA` and `addedAmountB` are greater than 0:

```
if (addedAmountA > 0) tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
if (addedAmountB > 0) tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );
```



# [G-22] Use `delete` instead of assigning mapping variable to `0`

[File: PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L54)
```
_arbitrageProfits[ poolIDs[i] ] = 0;
```

[File: StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L301)
```
cooldowns[i] = 0;
```


# [G-23] Loops which iterates only a few times can be coded directly

[File: ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L115)
```
for( uint256 i = 0; i < 8; i++ )
```

The loop executes only 7 times and is very short. It might be reasonable to use the code inside the loop directly, to save gas from the loop initialization and checking `i < 8` conditions on every loop iteration.


# [G-24] Cache state variables when reading them more than once

[File: Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L221)
```
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;
```

We read `_userDeposits[msg.sender][token]` three times (one in `require` and twice to perform `-=`). We should cache its value:

```
    	uint256 _userDepositsCached = _userDeposits[msg.sender][token]
    	require(_userDepositsCached >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] = _userDepositsCached - amount;
```

# [G-25] Change the order of `if`-`else` to avoid negation

`if ( !A ) do_X(); else do_Y();` should be changed to `if ( A ) do_Y(); else do_X();` to avoid negation in the first `if` condition.

[File: CoreUniswapFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreUniswapFeed.sol#L125)
```
        if ( ! weth_usdcFlipped )
        	return 10**36 / uniswapWETH_USDC;
        else
        	return uniswapWETH_USDC;
		}
```

can be changed to:

```
        if (  weth_usdcFlipped )
        	return uniswapWETH_USDC;
        else
        	return 10**36 / uniswapWETH_USDC;
		}
```

# [G-26] `else` is redundant when previous `if` returned

[File: CoreUniswapFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreUniswapFeed.sol#L125)
```
        if ( ! weth_usdcFlipped )
        	return 10**36 / uniswapWETH_USDC;
        else
        	return uniswapWETH_USDC;
		}
```

We can save opcode responsible for performing `else` condition. When first `if` didn't return, we can return immediately after it without `else`:

```
 if ( ! weth_usdcFlipped )
        	return 10**36 / uniswapWETH_USDC;
 return return uniswapWETH_USDC;
```

# [G-27] Use Yul assembly to calculate simple expressions

* In `PriceAggregator.sol`, in function `_aggregatePrices()`, we calculate the average: `uint256 averagePrice = ( priceA + priceB ) / 2;`. Since it's a simple expression, it can be easily optimized by being rewritten to Yul.

* In `PriceAggregator.sol`, function `_absoluteDifference()` is a simple expression (`abs(x-y)`) which can be easily optimized by being rewritten to Yul.


# [G-28] Do not calculate the length of array/mapping multiple of times

Calculating length of array/mapping uses a lot of gas. It's always better to calculate it once and cache the result in a local variable.
While bot-report reported issue related to array's length in the loop (which are calculated on every loop iteration) - there are some other places where the length of array should be cached.

[File: SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L84)
```
		uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length; // poolIDs.length is guaranteed to not be zero

		AddedReward[] memory addedRewards = new AddedReward[]( poolIDs.length );
		for( uint256 i = 0; i < addedRewards.length; i++ )
```

Since `poolIDs.length == addedRewards.length`, we can cache it even before the loop:

```
		uint256 poolIDsLength = poolIDs.length;
		uint256 amountPerPool = liquidityBootstrapAmount / poolIDsLength; // poolIDs.length is guaranteed to not be zero

		AddedReward[] memory addedRewards = new AddedReward[]( poolIDsLength );
		for( uint256 i = 0; i < poolIDsLength; i++ )
```

[File: SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L120)
```
require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

[...]
		for( uint256 i = 0; i < poolIDs.length; i++ )
```

Bot-report suggested to cache it before the loop, but we can do it even sooner:

```
uint256 poolIDsLength = poolIDs.length;
require( poolIDsLength == profitsForPools.length, "Incompatible array lengths" );

[...]
		for( uint256 i = 0; i < poolIDsLength; i++ )
```

# [G-29] Move `require` on top of function

It's recommened to have `require` statements as high as possible in the function's code. When `require` reverts, we won't waste gas on the code before that `require`.

[File: Staking.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L200)
```
		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
```

We can move `require` above `uint256 minUnstakePercent = stakingConfig.minUnstakePercent();`. If any of these `require` statemens reverted, we wouldn't waste gas on executing `stakingConfig.minUnstakePercent()`.

```
		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();

		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );

		uint256 minUnstakePercent = stakingConfig.minUnstakePercent();
```

# [G-30] Use do-while loops instead of for-loops

Most of the `for`-loops implements a simple iteration. We can easily change them to `do-while` loops which use less gas

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
./src/stable/CollateralAndLiquidity.sol:321:                    for ( uint256 i = startIndex; i <= endIndex; i++ )
./src/stable/CollateralAndLiquidity.sol:338:            for ( uint256 i = 0; i < count; i++ )
```


