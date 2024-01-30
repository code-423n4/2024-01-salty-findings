# Gas Optimizations

| Issue                                                                                                                                                                                                                                                                                                                                                                                                                                          | Total Gas Saved |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------: |
| [[G-01] **In `DAO.sol` refactor `finalizeBallot` function , `_finalizeParameterBallot`, `_finalizeApprovalBallot` and `_finalizeTokenWhitelisting` internal functions to avoid extra external calls.(Total Gas Saved ~7000 GAS)**](#g-01-in-daosol-refactor-finalizeballot-function--_finalizeparameterballot-_finalizeapprovalballot-and-_finalizetokenwhitelisting-internal-functions-to-avoid-extra-external-callstotal-gas-saved-7000-gas) |    **~7000**    |
| [[G-02] **State variables can be packed into fewer storage slots (saves ~30000 Gas)**](#g-02-state-variables-can-be-packed-into-fewer-storage-slots-saves-30000-gas)                                                                                                                                                                                                                                                                           |   **~30000**    |
| [[G-03] **Change the order of `modifier` check in `functions` can save upto 23.9k gas per instance half of the time(Total Total Gas Saved ~119500)**](#g-03-change-the-order-of-modifier-check-in-functions-can-save-upto-239k-gas-per-instance-half-of-the-timetotal-total-gas-saved-119500)                                                                                                                                                  |   **~119500**   |
| [[G-04] **Use alternative gas efficient approach to check that `string` is empty or not(Total Gas Saved ~250)**](#g-04-use-alternative-gas-efficient-approach-to-check-that-string-is-empty-or-nottotal-gas-saved-250)                                                                                                                                                                                                                         |    **~250**     |
| [[G-05] **Use existed immutable variable instead of fetching them from external call(Total Gas Saved ~2600)**](#g-05-use-existed-immutable-variable-instead-of-fetching-them-from-external-calltotal-gas-saved-2600)                                                                                                                                                                                                                           |    **~2600**    |
| [[G-06] **Make the variables immutable and `remove` the function since `setContracts` function called only once (Total Gas Saved ~4200 GAS)**](#g-06-make-the-variables-immutable-and-remove-the-function-since-setcontracts-function-called-only-once-total-gas-saved-4200-gas)                                                                                                                                                               |    **~4200**    |
| [[G-07] **External calls should be cached instead of re-calling the same function in the same transaction(Total Gas Saved ~3250 GAS)**](#g-07-external-calls-should-be-cached-instead-of-re-calling-the-same-function-in-the-same-transactiontotal-gas-saved-3250-gas)                                                                                                                                                                         |    **~3250**    |
| [[G-08] **Unnecessary `nonReentrant` modifier check can be removed from some functions(Total Gas Saved ~72000 GAS)**](#g-08-unnecessary-nonreentrant-modifier-check-can-be-removed-from-some-functionstotal-gas-saved-72000-gas)                                                                                                                                                                                                               |   **~72000**    |
| [[G-09] **Function calls in same function should be cached instead of re-calling it(Total Gas Saved ~120 Gas)**](#g-09-function-calls-in-same-function-should-be-cached-instead-of-re-calling-ittotal-gas-saved-120-gas)                                                                                                                                                                                                                       |    **~120**     |
| [[G-10] **Refactor code in `Staking::calculateUnstake` function.So that it can save `external` calls if any `require` reverts. Can save 5k most of the time and 10k some time. (Total Gas Saved ~5000)**](#g-10-refactor-code-in-stakingcalculateunstake-functionso-that-it-can-save-external-calls-if-any-require-reverts-can-save-5k-most-of-the-time-and-10k-some-time-total-gas-saved-5000)                                                |    **~5000**    |
| [[G-11] **Change `require` statements order to fail early with less gas if going to fails. Saving `18900 GAS` half of the times.**](#g-11-change-require-statements-order-to-fail-early-with-less-gas-if-going-to-fails-saving-18900-gas-half-of-the-times)                                                                                                                                                                                    |   **~18900**    |
| [[G-12] **Change `require` statements order to avoid `external calls` and saves `~31300 Gas` half of the times.**](#g-12-change-require-statements-order-to-avoid-external-calls-and-saves-31300-gas-half-of-the-times)                                                                                                                                                                                                                        |   **~31300**    |
| [[G-13] **Use already cached variable instead of re-reading from storage(Total Gas Saved ~200 GAS)**](#g-13-use-already-cached-variable-instead-of-re-reading-from-storagetotal-gas-saved-200-gas)                                                                                                                                                                                                                                             |    **~200**     |
| [[G-14] **Make The Variables Outside The Loop To Save Gas**](#g-14-make-the-variables-outside-the-loop-to-save-gas)                                                                                                                                                                                                                                                                                                                            |        -        |
| [[G-15] **Unnecessary `require` check can be `removed` (Total Gas Saved ~26 Gas)**](#g-15-unnecessary-require-check-can-be-removed-total-gas-saved-26-gas)                                                                                                                                                                                                                                                                                     |     **~26**     |
| [[G-16] **Use `storage` instead of `memory` for `struct/array`(Total Gas Saved ~14700 Gas)**](#g-16-use-storage-instead-of-memory-for-structarraytotal-gas-saved-14700-gas)                                                                                                                                                                                                                                                                    |   **~14700**    |

## [G-01] In `DAO.sol` refactor `finalizeBallot` function , `_finalizeParameterBallot`, `_finalizeApprovalBallot` and `_finalizeTokenWhitelisting` internal functions to avoid extra external calls.(Total Gas Saved ~7000 GAS)

Since external calls cost extra gas. So we should avoid them calling again if it is not necessary.

#### Total Gas Saved ~7000 GAS

In `finalizeBallot` function `proposals.ballotForID(ballotID)` is already called so we can avoid calling this function again in `_finalizeParameterBallot`, `_finalizeApprovalBallot` and `_finalizeTokenWhitelisting` internal functions which were also called in `finalizeBallot` only
proposals.ballotForID(ballotID)`is called also in each internal function but only one internal function will be called from`finalizeBallot`due if-else conditions. 
So overall we can avoid one `proposals.ballotForID(ballotID)` extra function call by passing`ballot`as parameter to these internal functions we can avoid that call again in these internal functions.

Again calling same `proposals.ballotForID(ballotID)` with same `ballotID` will cost **~7000 GAS**. Using `ballot` as param we can save almost **~7000 GAS** each time when`finalizeBallot` function will be called.

### Pass `ballot` as parameter to these internal functions

```solidity
File : src/dao/DAO.sol

113:    function _finalizeParameterBallot( uint256 ballotID ) internal
		{
		Ballot memory ballot = proposals.ballotForID(ballotID);


219:    function _finalizeApprovalBallot( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
			Ballot memory ballot = proposals.ballotForID(ballotID);


235:	function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
			// The ballot is approved. Any reversions below will allow the ballot to be attemped to be finalized later - as the ballot won't be finalized on reversion.
			Ballot memory ballot = proposals.ballotForID(ballotID);



278:  function finalizeBallot( uint256 ballotID ) external nonReentrant
		{
		// Checks that ballot is live, and minimumEndTime and quorum have both been reached
		require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );

		Ballot memory ballot = proposals.ballotForID(ballotID);

		if ( ballot.ballotType == BallotType.PARAMETER )
			_finalizeParameterBallot(ballotID);
		else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
			_finalizeTokenWhitelisting(ballotID);
		else
			_finalizeApprovalBallot(ballotID);
		}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L113C1-L115C58,
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L219C1-L221C47,
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L235C1-L240C59,
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L278C1-L291C4

**Recommended Mitigation steps:**

```diff
File : src/dao/DAO.sol

-   function _finalizeParameterBallot( uint256 ballotID ) internal
+   function _finalizeParameterBallot( uint256 ballotID, Ballot memory ballot) internal
		{
-		Ballot memory ballot = proposals.ballotForID(ballotID);




-    function _finalizeApprovalBallot( uint256 ballotID ) internal
+    function _finalizeApprovalBallot( uint256 ballotID, Ballot memory ballot) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
-			Ballot memory ballot = proposals.ballotForID(ballotID);


-	function _finalizeTokenWhitelisting( uint256 ballotID ) internal
+	function _finalizeTokenWhitelisting( uint256 ballotID, Ballot memory ballot) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
-			// The ballot is approved. Any reversions below will allow the ballot to be attemped to be finalized later - as the ballot won't be finalized on reversion.
-			Ballot memory ballot = proposals.ballotForID(ballotID);



  function finalizeBallot( uint256 ballotID ) external nonReentrant
		{
		// Checks that ballot is live, and minimumEndTime and quorum have both been reached
		require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );

		Ballot memory ballot = proposals.ballotForID(ballotID);

		if ( ballot.ballotType == BallotType.PARAMETER )
-			_finalizeParameterBallot(ballotID);
+			_finalizeParameterBallot(ballotID, ballot);
		else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
-			_finalizeTokenWhitelisting(ballotID);
+			_finalizeTokenWhitelisting(ballotID, ballot);
		else
-			_finalizeApprovalBallot(ballotID);
+			_finalizeApprovalBallot(ballotID, ballot);
		}

```

## [G-02] State variables can be packed into fewer storage slots (saves ~30000 Gas)

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

### Total `SAVE: ~30000 GAS, 15 Storage SLOT`

### `bootstrappingRewards`, `percentPolRewardsBurned`, `baseBallotQuorumPercentTimes1000`, `ballotMinimumDuration`, `requiredProposalPercentStakeTimes1000`, `maxPendingTokensForWhitelisting`, `arbitrageProfitsPercentPOL` and `upkeepRewardPercent` can be packed in single SLOT :`SAVES 14000 GAS`, `7 SLOT`

1. **`bootstrappingRewards`** This can be reduced to uint96(can hold up to **~7.9e28**) because `bootstrappingRewards` maximum range is between 50k-500k ether (50e18 - 500e18) so `uint96` is more than sufficient to hold this value.

2. **`percentPolRewardsBurned`** This can be reduced to uint8(max value is 255) because `percentPolRewardsBurned` max value is 50 so `uint8` is more than sufficient to hold this value.

3. **`baseBallotQuorumPercentTimes1000`** This can be reduced to uint16(max value is 65535) because max value of `baseBallotQuorumPercentTimes1000` can be 10000 so `uint16` is more than sufficient to hold this value.

4. **`ballotMinimumDuration`** This can be reduced to uint32(max value is 4,294,967,295) because max value of `ballotMinimumDuration` can be 14 days(12,09,600 in seconds) so `uint32` is more than sufficient to hold this value.

5. **`requiredProposalPercentStakeTimes1000`**: This can be reduced to uint16(max value is 65535) because max value of `requiredProposalPercentStakeTimes1000` can be 2000 so `uint16` is more than sufficient to hold this value.

6. **`maxPendingTokensForWhitelisting`** This can be reduced to uint8(max value is 255) because max value of `maxPendingTokensForWhitelisting` can be 12 so `uint8` is more than sufficient to hold this value.

7. **`arbitrageProfitsPercentPOL`** This can be reduced to uint8(max value is 255) because max value of `arbitrageProfitsPercentPOL` can be 45 so `uint8` is more than sufficient to hold this value.

8. **`upkeepRewardPercent`** This can be reduced to uint8(max value is 255) because max value of `upkeepRewardPercent` can be 10 so `uint8` is more than sufficient to hold this value.

By adding all of them 96+8+16+32+16+8+8+8 = 192 ie. 24 bytes still less than 32 bytes so they can fit into one slot. Saving 7 storage slots.

```solidity
File : src/dao/DAOConfig.sol

      // Range: 50k ether to 500k ether with an adjustment of 50k ether
      uint256 public bootstrappingRewards = 200000 ether;


	// Range: 25% to 75% with an adjustment of 5%
        uint256 public percentPolRewardsBurned = 50;


	// Range: 5% to 20% with an adjustment of 1%
	uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000;


	// Range: 3 to 14 days with an adjustment of 1 day
	uint256 public ballotMinimumDuration = 10 days;


	// Range: 0.10% to 2% with an adjustment of 0.10%
	uint256 public requiredProposalPercentStakeTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier


	// Range: 3 to 12 with an adjustment of 1
	uint256 public maxPendingTokensForWhitelisting = 5;


	// Range: 15% to 45% with an adjustment of 5%
	uint256 public arbitrageProfitsPercentPOL = 20;


	// Range: 1% to 10% with an adjustment of 1%
	uint256 public upkeepRewardPercent = 5;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L23C1-L61C41

**Recommended Mitigation steps:**

```diff
File : src/dao/DAOConfig.sol

    // Range: 50k ether to 500k ether with an adjustment of 50k ether
-	uint256 public bootstrappingRewards = 200000 ether;
+	uint96 public bootstrappingRewards = 200000 ether;

	// For rewards distributed to the DAO, the percentage of SALT that is burned with the remaining staying in the DAO for later use.
	// Range: 25% to 75% with an adjustment of 5%
-    uint256 public percentPolRewardsBurned = 50;
+    uint8 public percentPolRewardsBurned = 50;


	// Range: 5% to 20% with an adjustment of 1%
-	  uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000; // Default 10% of the total amount   of SALT staked with a 1000x multiplier
+	uint16 public baseBallotQuorumPercentTimes1000 = 10 * 1000; // Default 10% of the total amount of SALT staked with a 1000x multiplier

	// How many days minimum a ballot has to exist before it can be taken action on.
	// Action will only be taken if it has the required votes and quorum to do so.
	// Range: 3 to 14 days with an adjustment of 1 day
-	uint256 public ballotMinimumDuration = 10 days;
+	uint32 public ballotMinimumDuration = 10 days;

	// The percent of staked SALT that a user has to have to make a proposal
	// Range: 0.10% to 2% with an adjustment of 0.10%
-	uint256 public requiredProposalPercentStakeTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier
+	uint16 public requiredProposalPercentStakeTimes1000 = 500;  // Defaults to 0.50% with a 1000x multiplier

	// The maximum number of tokens that can be pending for whitelisting at any time.
	// Range: 3 to 12 with an adjustment of 1
-	uint256 public maxPendingTokensForWhitelisting = 5;
+	uint8 public maxPendingTokensForWhitelisting = 5;

	// The share of the WETH arbitrage profits that are sent to the DAO to form SALT/USDS Protocol Owned Liquidity
	// Range: 15% to 45% with an adjustment of 5%
-	uint256 public arbitrageProfitsPercentPOL = 20;
+	uint8 public arbitrageProfitsPercentPOL = 20;

	// The share of the WETH arbitrage profits sent to the DAO that are sent to the caller of DAO.performUpkeep()
	// Range: 1% to 10% with an adjustment of 1%
-	uint256 public upkeepRewardPercent = 5;
+	uint8 public upkeepRewardPercent = 5;

```

### `maximumWhitelistedPools` and `maximumInternalSwapPercentTimes1000` can be packed in single slot :`SAVES 2000 GAS`, `1 SLOT`

1. **`maximumWhitelistedPools`** This can be reduced to uint8(max value is 255) because `maximumWhitelistedPools` max value is 100 so uint8 is more than sufficient to hold this value.

2. **`maximumInternalSwapPercentTimes1000`** This can be reduced to uint16(max value is 65535) because max value of `maximumInternalSwapPercentTimes1000` can be 2000 so uint16 is more than sufficient to hold this value.

```solidity
File : src/pools/PoolsConfig.sol

    // Range: 20 to 100 with an adjustment of 10
	uint256 public maximumWhitelistedPools = 50;

	// For swaps internal to the protocol, the maximum swap size as a percent of the reserves.
	// Range: .25 to 2% with an adjustment of .25%
	uint256 public maximumInternalSwapPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L36C2-L41C105

**Recommended Mitigation steps:**

```diff
File : src/pools/PoolsConfig.sol

    // Range: 20 to 100 with an adjustment of 10
-	uint256 public maximumWhitelistedPools = 50;
+	uint8 public maximumWhitelistedPools = 50;

	// For swaps internal to the protocol, the maximum swap size as a percent of the reserves.
	// Range: .25 to 2% with an adjustment of .25%
-	uint256 public maximumInternalSwapPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier
+	uint16 public maximumInternalSwapPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier
```

### `priceFeedModificationCooldownExpiration`, `maximumPriceFeedPercentDifferenceTimes1000` and `priceFeedModificationCooldown` can be packed in a single slot `SAVES 4000 GAS`, `2 SLOT`

1. **`priceFeedModificationCooldownExpiration`** This can be reduced to uint64. The precise number of years that a uint64 type can safely accommodate is precisely 18,446,744,073,709,551,615 years.

2. **`maximumPriceFeedPercentDifferenceTimes1000`** This can be reduced to uint32(max value is 4,294,967,295) because max value of `maximumPriceFeedPercentDifferenceTimes1000` can be 7000 so uint32 is more than sufficient to hold this value.

3. **`priceFeedModificationCooldown`** This can be reduced to uint32(max value is 4,294,967,295) because max value of `priceFeedModificationCooldown` can be 45 Days(38,88,000 second) so uint32 is more than sufficient to hold this value.

```solidity
File : src/price_feed/PriceAggregator.sol
    // The next time at which setPriceFeed can be called
	uint256 public priceFeedModificationCooldownExpiration;

	// Range: 1% to 7% with an adjustment of .50%
	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

	// Range: 30 to 45 days with an adjustment of 5 days
	uint256 public priceFeedModificationCooldown = 35 days;
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L23C2-L34C57

**Recommended Mitigation steps:**

```diff
File : src/price_feed/PriceAggregator.sol

    // The next time at which setPriceFeed can be called
-	uint256 public priceFeedModificationCooldownExpiration;
+	uint64 public priceFeedModificationCooldownExpiration;

	// Range: 1% to 7% with an adjustment of .50%
-	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier
+	uint32 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

	// Range: 30 to 45 days with an adjustment of 5 days
-	uint256 public priceFeedModificationCooldown = 35 days;
+	uint32 public priceFeedModificationCooldown = 35 days;


```

### `rewardPercentForCallingLiquidation`, `maxRewardValueForCallingLiquidation`, `minimumCollateralValueForBorrowing`, `initialCollateralRatioPercent`, `minimumCollateralRatioPercent` and `percentArbitrageProfitsForStablePOL` can be packed in single slot `SAVES 10000 GAS`, `5 SLOT`

1. **`rewardPercentForCallingLiquidation`** This can be reduced to uint8(max value is 255) because `rewardPercentForCallingLiquidation` max value is 10 so uint8 is more than sufficient to hold this value.

2. **`maxRewardValueForCallingLiquidation`** This can be reduced to uint96(can hold up to ~7.9e28) because `maxRewardValueForCallingLiquidation` maximum range is between 100-1000 ether (100e18-500e18) so uint96 is more than sufficient to hold this value.

3. **`minimumCollateralValueForBorrowing`** This can be reduced to uint96(can hold up to ~7.9e28) because `minimumCollateralValueForBorrowing` maximum range is between 1000-5000 ether (1000e18-5000e18) so uint96 is more than sufficient to hold this value.

4. **`initialCollateralRatioPercent`** This can be reduced to uint16(max value is 65535) because max value of `initialCollateralRatioPercent` can be 300 so uint16 is more than sufficient to hold this value.

5. **`minimumCollateralRatioPercent`** This can be reduced to uint8(max value is 255) because `minimumCollateralRatioPercent` max value is 120 so uint8 is more than sufficient to hold this value.

6. **`percentArbitrageProfitsForStablePOL`** This can be reduced to uint8(max value is 255) because `percentArbitrageProfitsForStablePOL` max value is 10 so uint8 is more than sufficient to hold this value.

```solidity
File : src/stable/StableConfig.sol

19:	// Range: 5 to 10 with an adjustment of 1
    uint256 public rewardPercentForCallingLiquidation = 5;

	// Range: 100 to 1000 with an adjustment of 100 ether
    uint256 public maxRewardValueForCallingLiquidation = 500 ether;

	// Range: 1000 to 5000 with an adjustment of 500 ether
    uint256 public minimumCollateralValueForBorrowing = 2500 ether;

    // Range: 150 to 300 with an adjustment of 25
    uint256 public initialCollateralRatioPercent = 200;

	// Range: 110 to 120 with an adjustment of 1
	uint256 public minimumCollateralRatioPercent = 110;

	// Range: 1 to 10 with an adjustment of 1
44:	uint256 public percentArbitrageProfitsForStablePOL = 5;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L19C1-L44C57

**Recommended Mitigation steps:**

```diff
File : src/stable/StableConfig.sol

	// Range: 5 to 10 with an adjustment of 1
-    uint256 public rewardPercentForCallingLiquidation = 5;
+    uint8 public rewardPercentForCallingLiquidation = 5;

	// Range: 100 to 1000 with an adjustment of 100 ether
-    uint256 public maxRewardValueForCallingLiquidation = 500 ether;
+    uint96 public maxRewardValueForCallingLiquidation = 500 ether;

	// Range: 1000 to 5000 with an adjustment of 500 ether
-    uint256 public minimumCollateralValueForBorrowing = 2500 ether;
+    uint96 public minimumCollateralValueForBorrowing = 2500 ether;

    // Range: 150 to 300 with an adjustment of 25
-    uint256 public initialCollateralRatioPercent = 200;
+    uint16 public initialCollateralRatioPercent = 200;

	// Range: 110 to 120 with an adjustment of 1
-	uint256 public minimumCollateralRatioPercent = 110;
+	uint8 public minimumCollateralRatioPercent = 110;

	// Range: 1 to 10 with an adjustment of 1
-	uint256 public percentArbitrageProfitsForStablePOL = 5;
+	uint8 public percentArbitrageProfitsForStablePOL = 5;

```

## [G-03] Change the order of `modifier` check in `functions` can save upto 23.9k gas per instance half of the time(Total Total Gas Saved ~119500)

**Gas per instance: ~23900**,

**Total Instances: 5**,

**Total Gas Saved: ~23900 \* 5 = ~119500**

### Change the order of `modifier` first check the `deadline` because `ensureNotExpired` check only takes `~100 Gas` and `nonReentrant` check consume almost `~24000 gas` if we check `deadline` first we can `save up to ~23900 Gas` when both modifier is reverting or only `ensureNotExpired` failing.

The function incorporates two modifiers, `nonReentrant` and `ensureNotExpired(deadline)`. An optimization suggestion is made to reorder these modifiers for potential gas savings. The proposed order aims to maximize gas efficiency, considering scenarios where the deadline has expired or the second condition is more gas-consuming in the event of failure.

### `SAVES:~23900 GAS` if `ensureNotExpired(deadline)` fails or both modifier fails

```diff
File : src/pools/Pools.sol

- function swap( IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
+ function swap( IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external ensureNotExpired(deadline) nonReentrant  returns (uint256 swapAmountOut)
		{

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L366C2-L367C4

### `SAVES:~23900 GAS` if ensureNotExpired(deadline) fails or both modifier fails

```diff
File : src/pools/Pools.sol

- function depositSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
+ function depositSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external ensureNotExpired(deadline) nonReentrant returns (uint256 swapAmountOut)
		{
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L382C2-L383C4

### `SAVES:~23900 GAS` if ensureNotExpired(deadline) fails or both modifier fails

```diff
File : src/pools/Pools.sol

- function depositDoubleSwapWithdraw( IERC20 swapTokenIn, IERC20 swapTokenMiddle, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
+ function depositDoubleSwapWithdraw( IERC20 swapTokenIn, IERC20 swapTokenMiddle, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external ensureNotExpired(deadline) nonReentrant  returns (uint256 swapAmountOut)
		{
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L395C2-L396C4

### `SAVES:~23900 GAS` if ensureNotExpired(deadline) fails or both modifier fails

```diff
File : src/stable/CollateralAndLiquidity.sol

- function depositCollateralAndIncreaseShare( uint256 maxAmountWBTC, uint256 maxAmountWETH, uint256 minLiquidityReceived, uint256 deadline, bool useZapping ) external nonReentrant ensureNotExpired(deadline)  returns (uint256 addedAmountWBTC, uint256 addedAmountWETH, uint256 addedLiquidity)
+ function depositCollateralAndIncreaseShare( uint256 maxAmountWBTC, uint256 maxAmountWETH, uint256 minLiquidityReceived, uint256 deadline, bool useZapping ) external ensureNotExpired(deadline)  nonReentrant  returns (uint256 addedAmountWBTC, uint256 addedAmountWETH, uint256 addedLiquidity)
		{

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L70C2-L71C4

### `SAVES:~23900 GAS` if ensureNotExpired(deadline) fails or both modifier fails

```diff
File : src/stable/CollateralAndLiquidity.sol

- function withdrawCollateralAndClaim( uint256 collateralToWithdraw, uint256 minReclaimedWBTC, uint256 minReclaimedWETH, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 reclaimedWBTC, uint256 reclaimedWETH)
+ function withdrawCollateralAndClaim( uint256 collateralToWithdraw, uint256 minReclaimedWBTC, uint256 minReclaimedWETH, uint256 deadline ) external ensureNotExpired(deadline) nonReentrant returns (uint256 reclaimedWBTC, uint256 reclaimedWETH)
		{


```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L80C5-L81C4

## [G-04] Use alternative gas efficient approach to check that `string` is empty or not(Total Gas Saved ~250)

### `SAVES: ~250 GAS`

After optimizing code it saves `266 gas` on passing and `248 gas` on failing require condition.
_Gas Calculated on optimizer runs 20000._

```solidity
File : src/dao/Proposals.sol

249:   function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
250: {
251: require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" ); //@audit gas update the logic to check this

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L249C2-L251C125

**Recommended Mitigation steps:**

**This refined version maintains the original functionality of ensuring that `newWebsiteURL` is not empty.**

```diff
File : src/dao/Proposals.sol

   function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)		{
-		require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );

+	require(bytes(newWebsiteURL).length != 0,"newWebsiteURL cannot be empty");

```

## [G-05] Use existed immutable variable instead of fetching them from external call(Total Gas Saved ~2600)

### Using immutable var. `salt` instead of doing external call `exchangeConfig.salt()` can save **~650 GAS** every time this external call being called.

**Total GAS Saved : ~2600 GAS in 4 instances**

`exchangeConfig.salt()` is already set in the constructor as `salt = exchangeConfig.salt();` immutable salt variable, so use `salt` directly instead of using external call to fetch same `salt`

### Saves 2 External calls since `exchangeConfig.salt()` called two time in `_executeApproval` function Total Gas Saved ~ 1300 Gas

_~650 per instance_

```solidity
File : src/dao/DAO.sol

155: function _executeApproval( Ballot memory ballot ) internal
		{
...
166:		else if ( ballot.ballotType == BallotType.SEND_SALT )
			{
...
@>			if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
				{
@>				IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );

				emit SaltSent(ballot.address1, ballot.number1);
				}
176:			}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L155C2-L176C5

**Recommended Mitigation steps:**

```diff
File : src/dao/DAO.sol

function _executeApproval( Ballot memory ballot ) internal
		{
		if ( ballot.ballotType == BallotType.UNWHITELIST_TOKEN )
			{
			// All tokens are paired with both WBTC and WETH so unwhitelist those pools
			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );

			emit UnwhitelistToken(IERC20(ballot.address1));
			}

		else if ( ballot.ballotType == BallotType.SEND_SALT )
			{
			// Make sure the contract has the SALT balance before trying to send it.
			// This should not happen but is here just in case - to prevent approved proposals from reverting on finalization.
-			if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
+			if ( salt.balanceOf(address(this)) >= ballot.number1 )
				{
-				IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );
+				IERC20(salt).safeTransfer( ballot.address1, ballot.number1 );

				emit SaltSent(ballot.address1, ballot.number1);
				}
			}

```

### Saves 2 External calls of `exchangeConfig.salt()` in `_finalizeTokenWhitelisting` function it will save ~ 1300 Gas

_~650 per instance_

```solidity
File : src/dao/DAO.sol

235: function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
            uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
...
...

			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
			liquidityRewardsEmitter.addSALTRewards( addedRewards );

			emit WhitelistToken(IERC20(ballot.address1));
269:			}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L246C4-L269C5

**Recommended Mitigation steps:**

```diff
File : src/dao/DAO.sol

-          uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
+          uint256 saltBalance = salt.balanceOf( address(this) );
			require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

			// Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

			// All tokens are paired with both WBTC and WETH, so whitelist both pairings
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );

			// Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
			AddedReward[] memory addedRewards = new AddedReward[](2);
			addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
			addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

-			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
+			salt.approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
			liquidityRewardsEmitter.addSALTRewards( addedRewards );

			emit WhitelistToken(IERC20(ballot.address1));
			}

```

## [G-06] Make the variables immutable and `remove` the function since `setContracts` function called only once (Total Gas Saved ~4200 GAS)

Making var. immutable can save `Gcoldsload(2100 Gas)` every time when first time var. accessed in txn and 100 Gas of `Gwarmaccess` in same txn subsequent accessing of that variable.

Removing a function can also save deployments costs and one time access of that function since it was called only once.

### `dao` and `collateralAndLiquidity` in `Pools.sol` can be marked immutable saves ( ~4200 Gas)

The current `Pools` contract features a `setContracts` function designed to be called only once during deployment, assigning values to the `dao` and `collateralAndLiquidity` variables. For Gas efficiency it is advisable to declare these variables as `immutable`, assign them into `constructor` and the `setContracts` function can be `removed`. Also no need to inherit `Ownable` since `onlyOwner`only used here after that ownership renounced.

```solidity
File : src/pools/Pools.sol

39: IDAO public dao;
40:	ICollateralAndLiquidity public collateralAndLiquidity;
...
52:	constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig )
	ArbitrageSearch(_exchangeConfig)
	PoolStats(_exchangeConfig, _poolsConfig)
		{
		}
...
59:    // This will be called only once - at deployment time
	 function setContracts( IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
		{
		dao = _dao;
		collateralAndLiquidity = _collateralAndLiquidity;

		// setContracts can only be called once
		renounceOwnership();
67:		}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L39-L67

**Recommended Mitigation steps:**

```diff
File : src/pools/Pools.sol

- import "openzeppelin-contracts/contracts/access/Ownable.sol";

-	IDAO public dao;
-	ICollateralAndLiquidity public collateralAndLiquidity;
+	IDAO public immutable dao;
+	ICollateralAndLiquidity public immutable collateralAndLiquidity;

- constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig )
+ constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity)
	ArbitrageSearch(_exchangeConfig)
	PoolStats(_exchangeConfig, _poolsConfig)
		{
+		dao = _dao;
+		collateralAndLiquidity = _collateralAndLiquidity;
		}


-	// This will be called only once - at deployment time
-	function setContracts( IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
-		{
-		dao = _dao;
-		collateralAndLiquidity = _collateralAndLiquidity;
-
-		// setContracts can only be called once
-		renounceOwnership();
-		}

```

## [G-07] External calls should be cached instead of re-calling the same function in the same transaction(Total Gas Saved ~3250 GAS)

Since external calls cost extra gas. So we should avoid them calling again if it is not necessary.

### `exchangeConfig.accessManager()` is called multiple times in the `_executeApproval` function this call should be cached `SAVES: ~650 Gas`

```solidity
File : src/dao/DAO.sol

155: function _executeApproval( Ballot memory ballot ) internal
		{
...

...
		else if ( ballot.ballotType == BallotType.INCLUDE_COUNTRY )
			{
			excludedCountries[ ballot.string1 ] = false;

			emit GeoExclusionUpdated(ballot.string1, false, exchangeConfig.accessManager().geoVersion());
			}

		else if ( ballot.ballotType == BallotType.EXCLUDE_COUNTRY )
			{
			excludedCountries[ ballot.string1 ] = true;

@>			exchangeConfig.accessManager().excludedCountriesUpdated();

@>			emit GeoExclusionUpdated(ballot.string1, true, exchangeConfig.accessManager().geoVersion());
199:			}


```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L155C2-L199C96

**Recommended Mitigation steps:**

```diff
File : src/dao/DAO.sol

function _executeApproval( Ballot memory ballot ) internal
		{

		...

		else if ( ballot.ballotType == BallotType.INCLUDE_COUNTRY )
			{
			excludedCountries[ ballot.string1 ] = false;

-			emit GeoExclusionUpdated(ballot.string1, false, exchangeConfig.accessManager().geoVersion());
+			emit GeoExclusionUpdated(ballot.string1, false, _exchangeConfigAccessManager.geoVersion());
			}

		else if ( ballot.ballotType == BallotType.EXCLUDE_COUNTRY )
			{
+        IExchangeConfig _exchangeConfigAccessManager = exchangeConfig.accessManager();
			excludedCountries[ ballot.string1 ] = true;


-			exchangeConfig.accessManager().excludedCountriesUpdated();
+			_exchangeConfigAccessManager.excludedCountriesUpdated();

-			emit GeoExclusionUpdated(ballot.string1, true, exchangeConfig.accessManager().geoVersion());
+			emit GeoExclusionUpdated(ballot.string1, true, _exchangeConfigAccessManager.geoVersion());
			}

```

### `exchangeConfig.wbtc()` and `exchangeConfig.weth()` are called multiple times in the `_finalizeTokenWhitelisting` function these calls should be cached `SAVES: ~1300 GAS`

```solidity
File : src/dao/DAO.sol

235: function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
...
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );

			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
259:			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );


```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L254C1-L259C1

**Recommended Mitigation steps:**

```diff
File : src/dao/DAO.sol

 function _finalizeTokenWhitelisting( uint256 ballotID ) internal
		{
...
+        IExchangeConfig _exchangeConfigWbtc = exchangeConfig.wbtc();
+        IExchangeConfig _exchangeConfigWeth = exchangeConfig.weth();

-			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
-			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );
+			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), _exchangeConfigWbtc );
+			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), _exchangeConfigWeth );

-			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
-			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
+			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), _exchangeConfigWbtc );
+			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), _exchangeConfigWeth );

```

### `exchangeConfig.wbtc()` and `exchangeConfig.weth()` are called multiple times in the `proposeTokenUnwhitelisting` function these calls should be cached `SAVES: ~1300 GAS`

```solidity
File : src/dao/Proposals.sol

180:    function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
187:		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L180C2-L187C90

**Recommended Mitigation steps:**

```diff
File : src/dao/Proposals.sol

  function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
+        IExchangeConfig _exchangeConfigWbtc = exchangeConfig.wbtc();
+        IExchangeConfig _exchangeConfigWeth = exchangeConfig.weth();

-		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
+		require( poolsConfig.tokenHasBeenWhitelisted(token, _exchangeConfigWbtc, _exchangeConfigWeth), "Can only unwhitelist a whitelisted token" );
-		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
-		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
+		require( address(token) != address(_exchangeConfigWbtc), "Cannot unwhitelist WBTC" );
+		require( address(token) != address(_exchangeConfigWeth), "Cannot unwhitelist WETH" );
		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

```

## [G-08] Unnecessary `nonReentrant` modifier check can be removed from some functions(Total Gas Saved ~72000 GAS)

Removing `nonReentrant` modifier can save ~24000 Gas per instance. We can remove it from those functions where re-entrancy not possible.

### No need of `nonReentrant` check in `markBallotAsFinalized` function `SAVES: ~24000 GAS`

Since `exchangeConfig.dao()` is view function it return a address and it is only external call. dao in exchangeConfig is immutable variable. `exchangeConfig` is immutable here so only assigned by deployer of this contract. So re-entrancy is not possible we can safely remove `nonReentrant` modifier from here which can save upto 24000 Gas approx.

```solidity
File : src/dao/Proposals.sol

130: function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
		{
		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
    ...
		emit BallotFinalized(ballotID);
152:		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L130C2-L152C4

**Recommended Mitigation steps:**

```diff
File : src/dao/Proposals.sol

- function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
+ function markBallotAsFinalized( uint256 ballotID ) external
		{
		require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
    ...
		emit BallotFinalized(ballotID);
		}

```

### No need of `nonReentrant` check in `castVote` function `SAVES:~24000 GAS`

Explanation: The call to `staking.userShareForPool` is only external call here, and it is a view function.
The `userShareForPool` function only reads from a mapping and `staking` is immutable and assigned by the deployer of this contract,
making it immune to tampering by malicious contracts. Since the function is view-only,
meaning it does not modify the state, the `nonReentrant` modifier is not necessary in this context. We can safely remove it.

```solidity
File : src/dao/Proposals.sol

259:  function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{
...

		uint256 userVotingPower = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
277:    require( userVotingPower > 0, "Staked SALT required to vote" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L259C2-L277C66

**Recommended Mitigation steps:**

```diff
File : src/dao/Proposals.sol

-  function castVote( uint256 ballotID, Vote vote ) external nonReentrant
+  function castVote( uint256 ballotID, Vote vote ) external
		{
...

		uint256 userVotingPower = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
    require( userVotingPower > 0, "Staked SALT required to vote" );

```

### No need of `nonReentrant` check in `vote` function `SAVES:~24000 GAS`

The reentrancy guard can be safely omitted in this function, as the final operation is an external call,
posing no reentrancy risks. All state updates within the function have already been executed,
ensuring the integrity of the contract's state.

Additionally, `airdrop` is marked as immutable, and the contract instance is set by the deployer.
The immutability designation, combined with the deployer's assignment, enhances the security and reliability
of the operation, further justifying the removal of the reentrancy guard `nonReentrant`.

```solidity
File : src/launch/BootstrapBallot.sol

48:	function vote( bool voteStartExchangeYes, bytes calldata signature ) external nonReentrant
		{
	...

		// As the whitelisted user has retweeted the launch message and voted, they are authorized to the receive the airdrop.
		airdrop.authorizeWallet(msg.sender);
65:		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L48C1-L65C4

**Recommended Mitigation steps:**

```diff
File : src/launch/BootstrapBallot.sol

-	function vote( bool voteStartExchangeYes, bytes calldata signature ) external nonReentrant
+	function vote( bool voteStartExchangeYes, bytes calldata signature ) external
		{
	...

		// As the whitelisted user has retweeted the launch message and voted, they are authorized to the receive the airdrop.
		airdrop.authorizeWallet(msg.sender);
		}

```

## [G-09] Function calls in same function should be cached instead of re-calling it(Total Gas Saved ~120 Gas)

Function defined in contract should also be cached rather than re-calling that in same function.

**Note: Not in bot report**

### Result of `numberAuthorized()` function should be cached saves ~120 Gas

The function `numberAuthorized` is called 2 times within `allowClaiming`, resulting in 2 time state reads every time it's invoked.

To optimize gas usage, we should call the function once and store the result.

```solidity
File : src/launch/Airdrop.sol

56: function allowClaiming() external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
    	require( ! claimingAllowed, "Claiming is already allowed" );
@>		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");//@audit cache this function

    	// All users receive an equal share of the airdrop.
    	uint256 saltBalance = salt.balanceOf(address(this));
@>		saltAmountForEachUser = saltBalance / numberAuthorized();//@audit cache this

		// Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
		salt.approve( address(staking), saltBalance );

    	claimingAllowed = true;
70:    	}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L56C5-L70C7, https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L97-L100

```solidity

function numberAuthorized() public view returns (uint256)
    	{
    	return _authorizedUsers.length();
    	}

```

**Recommended Mitigation steps:**

**This refactoring optimizes gas usage by calling the `numberAuthorized` function once and storing the result in the variable `authorizedCount`, thereby eliminating redundant state reads within the `allowClaiming` function.**

```diff
File : src/launch/Airdrop.sol

 function allowClaiming() external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
    	require( ! claimingAllowed, "Claiming is already allowed" );
+       uint256 _numberAuthorized = numberAuthorized();
-		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+		require(_numberAuthorized > 0, "No addresses authorized to claim airdrop.");

    	// All users receive an equal share of the airdrop.
    	uint256 saltBalance = salt.balanceOf(address(this));
-		saltAmountForEachUser = saltBalance / numberAuthorized();
+		saltAmountForEachUser = saltBalance / numberAuthorized;

		// Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
		salt.approve( address(staking), saltBalance );

    	claimingAllowed = true;
    	}

```

## [G-10] Refactor code in `Staking::calculateUnstake` function.So that it can save `external` calls if any `require` reverts. Can save 5k most of the time and 10k some time. (Total Gas Saved ~5000)

`stakingConfig.maxUnstakeWeeks()` and `stakingConfig.minUnstakePercent()` are external calls that are reading state variables in from external contract. Which can take upto 5000 Gas approx. By refactoring the code we can ensure if 1st require fails then it should fails before other external call whose result not used in first condition. Similarly for 2nd require only that external call should be above whose result used in 2nd other will be after require. So if require fails it should consume as less gas as possible.

###

```solidity
File : src/staking/Staking.sol

198:    function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
...
204:		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
205:		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
...
212:		}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L198C2-L212C4

```diff
File : src/staking/Staking.sol

function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
		{
		uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
+		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
+		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

-		require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
-		require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );

		uint256 percentAboveMinimum = 100 - minUnstakePercent;
		uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;

		uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
    	return numerator / ( 100 * unstakeRange );
		}

```

## [G-11] Change `require` statements order to fail early with less gas if going to fails. Saving `18900 GAS` half of the times.

If a function has multiple require statements then they should be arranged in a increasing order of gas consuming to fail early as possible with less gas. So that most less consuming one should come at top if they are not necessary to be in order.
After that we should write more gas consuming require. and so on . So that if require fails then fail with less gas wasting. If both fails then 1st will revert with less gas. If initial fail fail with less gas.

So if two statements less consumer and more consumer.
4 conditions can happen

1.  both fails then less gas consumer fail first can save gas
2.  less gas consumer fails only still we will save gas not going to execute more gas consumer.
3.  If more consumer fails only then more gas will be consumed.
4.  If both passes good both will consume their gas
    So in above 4 conditions if we arrange require statements less gas consuming to more gas consuming one then in 1 and 2 condition ie .in half of the time the more gas will be saved by not executing more gas consuming require statement. But if we write more gas consumer 1st then more gas will be consumed in all 4 conditions So we should always write less gas consuming require statements first if it fails then more wont be executed and more gas consuming require's gas will be saved.

### In `finalizeBallot()` function first `require()` read `storage` variable `ballotFinalized` and second read `immutable` variable `completionTimestamp` we can swap both requires to `save 1 Gcoldsload (2100 gas)` half of the time.

```solidity
File : src/launch/BootstrapBallot.sol

69:     function finalizeBallot() external nonReentrant
70: 		{
71:		    require( ! ballotFinalized, "Ballot has already been finalized" );
72:		    require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L69C2-L72C83

**Recommended Mitigation steps:**

```diff
File : src/launch/BootstrapBallot.sol

    function finalizeBallot() external nonReentrant
		{
-		require( ! ballotFinalized, "Ballot has already been finalized" );
		require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
+		require( ! ballotFinalized, "Ballot has already been finalized" );

```

### In `addLiquidity` function first and second `require()` read `storage` variables `collateralAndLiquidity` ,`exchangeIsLive` and last three require read just constant and function params so we can change the order of require statements to `save 2 Gcoldsload (4200 gas)` most of the times.

```solidity
File : src/pools/Pools.sol

140: function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
		{
		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( exchangeIsLive, "The exchange is not yet live" );
		require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );

		require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
147:		require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L140C1-L147C86

**Recommended Mitigation steps:**

```diff
File : src/pools/Pools.sol

 function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
		{
-		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
-		require( exchangeIsLive, "The exchange is not yet live" );
		require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );

		require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
		require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );
+		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
+		require( exchangeIsLive, "The exchange is not yet live" );

```

### In function `removeLiquidity()` first `require()` read `storage` variable `collateralAndLiquidity` and second read function parameter `liquidityToRemove` so we can change these checks order to `save 1 Gcoldsload (2100 gas)` half of the time. Similarly we can change the order of other last two `require` to `save 2 Gcoldsload (4200 gas)` half of the time since in them, first require also reading from storage var. `reserves.reserve0` 2 times.ie 2 Gcoldsload can be saved half of the times. (Total Saved in `removeLiquidity()` ~6300 GAS) half of the times.

**Note: reserves.reserve0 should not be read two times 2nd time it should read reserves.reserve1. This is bug here which I reported separately**

```solidity
File : src/pools/Pools.sol

170:    function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
171:		{
172:		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
173:		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

187: require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

		// Switch reclaimed amounts back to the order that was specified in the call arguments so they make sense to the caller
		if (flipped)
			(reclaimedA,reclaimedB) = (reclaimedB,reclaimedA);

		require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

		// Send the reclaimed tokens to the user
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		emit LiquidityRemoved(tokenA, tokenB, reclaimedA, reclaimedB, liquidityToRemove);
200:		}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L170-L200

**Recommended Mitigation steps:**

```diff
File : src/pools/Pools.sol

     function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
		{
-		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
+		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );


-        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

		// Switch reclaimed amounts back to the order that was specified in the call arguments so they make sense to the caller
		if (flipped)
			(reclaimedA,reclaimedB) = (reclaimedB,reclaimedA);

		require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

+        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

		// Send the reclaimed tokens to the user
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		emit LiquidityRemoved(tokenA, tokenB, reclaimedA, reclaimedB, liquidityToRemove);
		}
```

### In `withdraw()` function first `require()` read `storage` variable `_userDeposits[msg.sender][token]`and second read function parameter `amount` and constant `DUST` from library we can change the order of these checks to `save 1 Gcoldsload (2100 gas)` half of the times.

```solidity
File : src/pools/Pools.sol

219:    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
220:    	{
221:    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
222:        require( amount > PoolUtils.DUST, "Withdraw amount too small");

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L219C1-L222C72

**Recommended Mitigation steps:**

```diff
File : src/pools/Pools.sol

    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
-    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");
+    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

```

### In `whitelistPool()` function first `require()` read two `storage` variables `_whitelist.length()`,`maximumWhitelistedPools` and second just compares function parameter tokenA and tokenB. So we can the order of these checks to `save 2 Gcoldsload (4200 gas)` half of the times.

```solidity
File : src/pools/PoolsConfig.sol

45:    function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
46:  		{
47: 		require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
48: 		require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L45C2-L48C75

**Recommended Mitigation steps:**

```diff
File : src/pools/PoolsConfig.sol

    function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
		{
-		require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
		require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
+		require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );

```

## [G-12] Change `require` statements order to avoid `external calls` and saves `~31300 Gas` half of the times.

Since external calls cost extra gas. So we should avoid them calling if it is not necessary.
To avoid cost more gas at the time of failing we should write more gas consuming external call contained require statement after the normal require statement or after the less gas consuming external call contained require statement if they are independent in flow write first that require which consumes less gas.

### In `_possiblyCreateProposal` function First check that `the user doesn't already have an active proposal` because if that check is false we can avoid `2 external function call`, storage reads and many opcodes. This is less gas costlier than it's above 2 requires. First can take 5k gas approx and 2nd one takes ~5k also while 3rd takes 2.1k which is less compare to both. So we can `save ~8000 Gas` 2/3rd of the times when any condition fails.

```solidity
File : src/dao/Proposals.sol

81: function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256
...
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
99:			}

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L81C2-L99C5

**Recommended Mitigation steps:**

```diff
File : src/dao/Proposals.sol

function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
		{
		require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

		// The DAO can create confirmation proposals which won't have the below requirements
		if ( msg.sender != address(exchangeConfig.dao() ) )
			{
+		// Make sure that the user doesn't already have an active proposal
+   require(! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time");
			// Make sure that the sender has the minimum amount of xSALT required to make the proposal
			uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

			require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

			uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
			require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );

-			// Make sure that the user doesn't already have an active proposal
-			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time");
			}

```

### In `proposeTokenUnwhitelisting` function Place the first require check in the last of all requires to avoid 2 gas expensive external function calls most of the times. Since others take almost 650 gas in each require (saves ~5000 Gas most of the times).

```solidity
File : src/dao/Proposals.sol

180:    function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
187:		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L180C2-L187C90

```diff
File : src/dao/Proposals.sol

    function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
-		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
+		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );

```

### In `authorizeWallet` function Place the first require check in the last to avoid 2 external function calls. First require takes almost 6k Gas while 2nd require takes almost 2.1k gas. So changing their order (saves ~4000 Gas half of the times.)

```solidity
File : src/launch/Airdrop.sol

46: function authorizeWallet( address wallet ) external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
    	require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );

		_authorizedUsers.add(wallet);
52:    	}
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L46C5-L52C7

**Recommended Mitigation steps:**

```diff
File : src/launch/Airdrop.sol

 function authorizeWallet( address wallet ) external
    	{
-    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
    	require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );

+    	require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );

		_authorizedUsers.add(wallet);
    	}

```

### In `allowClaiming` function place the less gas consuming require check first to save 1 external call. Since 1st one now consumes almost 5k gas calling external contracts storage var. Second and Third one consumes almost 2.1k gas each. So place the first at the third place change the order to save `~ 3000 Gas` half of the times.

```solidity
File : src/launch/Airdrop.sol

function allowClaiming() external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
    	require( ! claimingAllowed, "Claiming is already allowed" );
	require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L56C5-L60C80

**Recommended Mitigation steps:**

```diff
File : src/launch/Airdrop.sol

function allowClaiming() external
    	{
-    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
-    	require( ! claimingAllowed, "Claiming is already allowed" );
		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+    	require( ! claimingAllowed, "Claiming is already allowed" );
+    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
```

### In `proposeTokenUnwhitelisting` function Place the first `require` check in the last to avoid `1 external function call` reading from storage in their external contract ie in exchangeConfig. Which takes almost 5k gas. Place less gas consuming first only reading calldata params.`(saves ~5000 Gas) Half of the times.`

```solidity
File : src/rewards/SaltRewards.sol

117: function performUpkeep( bytes32[] calldata poolIDs, uint256[] calldata profitsForPools ) external
		{
		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
120:		require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L117C2-L120C85

**Recommended Mitigation steps:**

```diff
File : src/rewards/SaltRewards.sol

 function performUpkeep( bytes32[] calldata poolIDs, uint256[] calldata profitsForPools ) external
		{
-		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
		require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
+		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );

```

### In function `repayUSDS()` first `require()` call function, second read `state` variable and third read `function parameter`.First and second both take approx. 2.1k Gas but 3rd take very less gas compare to these. So after placing 3rd into first it Can save `~4200 Gas` some times and `~2100 Gas` most of the times.

```solidity
File : src/stable/CollateralAndLiquidity.sol

115:   function repayUSDS( uint256 amountRepaid ) external nonReentrant
116:		{
117:		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
118:		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
119:		require( amountRepaid > 0, "Cannot repay zero amount" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115C6-L119C59

```diff
File : src/stable/CollateralAndLiquidity.sol

    function repayUSDS( uint256 amountRepaid ) external nonReentrant
		{
+		require( amountRepaid > 0, "Cannot repay zero amount" );
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
-		require( amountRepaid > 0, "Cannot repay zero amount" );

```

### In this statement first `require()` read `storage` variable and second read `function parameter`. Change their order to save ~2100 Gas half of the times.

```solidity
File : src/stable/USDS.sol

42:     	require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
43: 		require( amount > 0, "Cannot mint zero USDS" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L42C2-L43C50

```diff
File : src/stable/USDS.sol

+		require( amount > 0, "Cannot mint zero USDS" );
	    require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
-		require( amount > 0, "Cannot mint zero USDS" );

```

### In function `_increaseUserShare()` first `require()` read `storage` variable and second read `function parameter`.Change their order to save `~2100 Gas` half of the times.

```solidity
File : src/staking/StakingRewards.sol

57:     function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
58: 		{
59: 		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
60: 		require( increaseShareAmount != 0, "Cannot increase zero share" );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L57C1-L60C69

```diff
File : src/staking/StakingRewards.sol

	function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
		{
+		require( increaseShareAmount != 0, "Cannot increase zero share" );
		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
-		require( increaseShareAmount != 0, "Cannot increase zero share" );

```

## [G-13] Use already cached variable instead of re-reading from storage(Total Gas Saved ~200 GAS)

### Use cached `reserve0` and `reserve1` in `_addLiquidity` function instead of `reserves.reserve0` and `reserves.reserve1` respectively `SAVES:200 GAS, 2 SLOADs` every time this function called

Since `reserve0` and `reserve1` is already cached if if condition at line 97 true then at line 100 and 101 in below code snippet we are reading state var. which are cached already so we can use that. Since if condition is true then return statement will return the function from if only. But that if condition fales then at line 125 and 126 we are reading same state variables which are already cached. So saving 2 SLOADs every time even if condition at line 97 true or false.`reserves.reserve0` and `reserves.reserve1` updated only once if if condition becomes false then at line 126,127 and if if condition true then at line 100 and 101 and return from there. Saving 2 Sloads Gwarmaccess 200 gas.

```solidity
File : src/pools/Pools.sol

90: function _addLiquidity( bytes32 poolID, uint256 maxAmount0, uint256 maxAmount1, uint256 totalLiquidity ) internal returns(uint256 addedAmount0, uint256 addedAmount1, uint256 addedLiquidity)
		{
		PoolReserves storage reserves = _poolReserves[poolID];
		uint256 reserve0 = reserves.reserve0;
		uint256 reserve1 = reserves.reserve1;

		// If either reserve is zero then consider the pool to be empty and that the added liquidity will become the initial token ratio
97:		if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
			{
			// Update the reserves
@>			reserves.reserve0 += uint128(maxAmount0);
@>101:			reserves.reserve1 += uint128(maxAmount1);

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
@>	    	reserves.reserve0 += uint128(addedAmount0);
@> 126:		reserves.reserve1 += uint128(addedAmount1);
```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L90C-L126

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L45C4-L49C5

## [G-14] Make The Variables Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements. By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost function execution.

_19 instances_

```solidity
File : src/arbitrage/ArbitrageSearch.sol

115:    for( uint256 i = 0; i < 8; i++ )
116:				{
1117:				uint256 midpoint = (leftPoint + rightPoint) >> 1;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L115C4-L117C54

```diff
File : src/arbitrage/ArbitrageSearch.sol

+       uint256 midpoint;
        for( uint256 i = 0; i < 8; i++ )
			{
-			uint256 midpoint = (leftPoint + rightPoint) >> 1;
+			midpoint = (leftPoint + rightPoint) >> 1;

```

```solidity
File : src/dao/Proposals.sol

423:    for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
424:			{
425:		uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
426:		uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
427:		uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423C3-L427C61

```diff
File : src/dao/Proposals.sol

+		uint256 ballotID;
+       uint256 yesTotal;
+       uint256 noTotal;
        for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
-		     uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
-		     uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
-		     uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];
+		     ballotID = _openBallotsForTokenWhitelisting.at(i);
+		     yesTotal = _votesCastForBallot[ballotID][Vote.YES];
+		     noTotal = _votesCastForBallot[ballotID][Vote.NO];

```

```solidity
File : src/pools/PoolStats.sol

for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];
			(IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81C3-L84C83

```diff
File : src/pools/PoolStats.sol

+       bytes32 poolID;
+       IERC20 arbToken2;
+       IERC20 arbToken3;
        for( uint256 i = 0; i < poolIDs.length; i++ )
			{
-			bytes32 poolID = poolIDs[i];
-			(IERC20 arbToken2, IERC20 arbToken3) = poolsConfig.underlyingTokenPair(poolID);
+			poolID = poolIDs[i];
+			(arbToken2, arbToken3) = poolsConfig.underlyingTokenPair(poolID);

```

```solidity
File : src/pools/PoolStats.sol

106:    for( uint256 i = 0; i < poolIDs.length; i++ )
107:			{
108:			// references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
109:			bytes32 poolID = poolIDs[i];
110:
111:			// Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
112:			uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L106C3-L112C60

```diff
File : src/pools/PoolStats.sol

+       bytes32 poolID;
+       uint256 arbitrageProfit;
        for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			// references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
-			bytes32 poolID = poolIDs[i];
+			poolID = poolIDs[i];

			// Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
-			uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
+			arbitrageProfit = _arbitrageProfits[poolID] / 3;

```

```solidity
File : src/rewards/SaltRewards.sol

64:		for( uint256 i = 0; i < addedRewards.length; i++ )
65:			{
66:			bytes32 poolID = poolIDs[i];
67:			uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L64C1-L67C92

```diff
File : src/rewards/SaltRewards.sol

+		bytes32 poolID;
+		uint256 rewardsForPool;
		for( uint256 i = 0; i < addedRewards.length; i++ )
			{
-			bytes32 poolID = poolIDs[i];
-			uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;
+			poolID = poolIDs[i];
+			rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;

```

```solidity
File : src/stable/CollateralAndLiquidity.sol

321:        for ( uint256 i = startIndex; i <= endIndex; i++ )
322:				{
323:				address wallet = _walletsWithBorrowedUSDS.at(i);
324:
325:				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
326:				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
327:
328:				// Determine minCollateral in terms of minCollateralValue
329:				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321C4-L329C97

```diff
File : src/stable/CollateralAndLiquidity.sol

+		address wallet;
+		uint256 minCollateralValue;
+		uint256 minCollateral;
        for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
-				address wallet = _walletsWithBorrowedUSDS.at(i);
+				wallet = _walletsWithBorrowedUSDS.at(i);

				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+				minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;

				// Determine minCollateral in terms of minCollateralValue
-				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
+				minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;

```

```solidity
File : src/staking/StakingRewards.sol

152:    for( uint256 i = 0; i < poolIDs.length; i++ )
153:			{
154:			bytes32 poolID = poolIDs[i];
155:
156:			uint256 pendingRewards = userRewardForPool( msg.sender, poolID );

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L152C3-L156C69

```diff
File : src/staking/StakingRewards.sol

+		bytes32 poolID;
+		uint256 pendingRewards;
        for( uint256 i = 0; i < poolIDs.length; i++ )
			{
-			bytes32 poolID = poolIDs[i];
+			poolID = poolIDs[i];

-			uint256 pendingRewards = userRewardForPool( msg.sender, poolID );
+			pendingRewards = userRewardForPool( msg.sender, poolID );

```

```solidity
File : src/staking/StakingRewards.sol

185:    for( uint256 i = 0; i < addedRewards.length; i++ )
186:			{
187:			AddedReward memory addedReward = addedRewards[i];
188:
189:			bytes32 poolID = addedReward.poolID;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185C3-L189C40

```diff
File : src/staking/StakingRewards.sol

+		AddedReward memory addedReward;
+		bytes32 poolID;
        for( uint256 i = 0; i < addedRewards.length; i++ )
			{
-			AddedReward memory addedReward = addedRewards[i];
+			addedReward = addedRewards[i];

-			bytes32 poolID = addedReward.poolID;
+			poolID = addedReward.poolID;

```

```solidity
File : src/staking/StakingRewards.sol

296:    for( uint256 i = 0; i < cooldowns.length; i++ )
297:			{
298:			uint256 cooldownExpiration = userInfo[ poolIDs[i] ].cooldownExpiration;

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L296C3-L298C75

```diff
File : src/staking/StakingRewards.sol

+		uint256 cooldownExpiration;
        for( uint256 i = 0; i < cooldowns.length; i++ )
			{
-			uint256 cooldownExpiration = userInfo[ poolIDs[i] ].cooldownExpiration;
+			cooldownExpiration = userInfo[ poolIDs[i] ].cooldownExpiration;

```

## [G-15] Unnecessary `require` check can be `removed` (Total Gas Saved ~26 Gas)

Small gas savings but optimizes the code with no downside.

### Since `end >=start` and `userUnstakes.length > end` than obviously `userUnstakes.length` will be higher than start. So removing 3rd require saves ~26 Gas.

There is no need to check `start < userUnstakes.length` in 3rd require. Since we have alraedy checked in first two requires that end is greater than or equal to start and `userUnstakes.length > end` then obviously userUnstakes.length will be greater than start also since `end >= start`. So no need to write extra require to check that.

```solidity
File : src/staking/Staking.sol

150: function unstakesForUser( address user, uint256 start, uint256 end ) public view returns (Unstake[] memory)
		{
        // Check if start and end are within the bounds of the array
        require(end >= start, "Invalid range: end cannot be less than start");

        uint256[] memory userUnstakes = _userUnstakeIDs[user];

        require(userUnstakes.length > end, "Invalid range: end is out of bounds");
158:    require(start < userUnstakes.length, "Invalid range: start is out of bounds");//@audit remove this check

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L150C2-L158C87

**Recommended Mitigation steps:**

```diff
File : src/staking/Staking.sol

 function unstakesForUser( address user, uint256 start, uint256 end ) public view returns (Unstake[] memory)
		{
        // Check if start and end are within the bounds of the array
        require(end >= start, "Invalid range: end cannot be less than start");

        uint256[] memory userUnstakes = _userUnstakeIDs[user];

        require(userUnstakes.length > end, "Invalid range: end is out of bounds");
-       require(start < userUnstakes.length, "Invalid range: start is out of bounds");

```

## [G-16] Use `storage` instead of `memory` for `struct/array`(Total Gas Saved ~14700 Gas)

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.
Since here in `castVote` function only 2 variables `ballot.ballotIsLive` and `ballot.ballotType` read at line 264 and 267 from `ballot` out of 9 variables from Ballot struct. Those 2 variables comes in one slot since they are tightly packed together because of writing together and their size. One is bool and one is enum . So from Ballot struct 7 slots are read unnecessarily at line 261. Causing extra ~14700 Gas to be wasted.
So direct read in storage pointer which can directly point to that storage location. And read only those 2 relevant variables. Cache if used one var. more than once but in this case it is used only one time so no need to cache.

**Note: Missed by bot report. This finding is present in bot report but this instance is not covered there. So I have reported it since it is significant finings saving `7 Gcoldsloads( ~14700 GAS)`. Really helpful for protocol**

```solidity
File : src/dao/Proposals.sol

259: function castVote( uint256 ballotID, Vote vote ) external nonReentrant
260:        {
261:        Ballot memory ballot = ballots[ballotID];//@audit use storage

263:    // Require that the ballot is actually live
@> 264:    require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

...     // Make sure that the vote type is valid for the given ballot
@> 267:    if ( ballot.ballotType == BallotType.PARAMETER )
            require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
        else // If a Ballot is not a Parameter Ballot, it is an Approval ballot
            require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
...
        emit VoteCast(msg.sender, ballotID, vote, userVotingPower);
293:    }

```

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L259C2-L293C4, https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/interfaces/IProposals.sol#L16C1-L30C3

```solidity
File : src/dao/interfaces/IProposals.sol

16: struct Ballot
    {
    uint256 ballotID;
    bool ballotIsLive;
    BallotType ballotType;
    string ballotName;
    address address1;
    uint256 number1;
    string string1;
    string description;

    // The earliest timestamp at which a ballot can end. Can be open longer if the quorum has not yet been reached for instance.
    uint256 ballotMinimumEndTime;
30: }
```

**Recommended Mitigation steps:**

```diff
File : src/dao/Proposals.sol

259: function castVote( uint256 ballotID, Vote vote ) external nonReentrant
260:        {
-261:       Ballot memory ballot = ballots[ballotID]
+261:       Ballot storage ballot = ballots[ballotID]

263:    // Require that the ballot is actually live
264:    require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

...     // Make sure that the vote type is valid for the given ballot
267:    if ( ballot.ballotType == BallotType.PARAMETER )
            require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
        else // If a Ballot is not a Parameter Ballot, it is an Approval ballot
            require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
...
        emit VoteCast(msg.sender, ballotID, vote, userVotingPower);
293:    }

```
