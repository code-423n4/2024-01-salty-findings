## Summary

- [[G-01] Replace `type(uint<n>).max` with constants](#g-01-replace-typeuintnmax-with-constants)
- [[G-02] State variables can be packed into fewer storage slots by truncating timestamp bytes](#g-02-state-variables-can-be-packed-into-fewer-storage-slots-by-truncating-timestamp-bytes)
- [[G-03] Order of initial checks in a function can be optimized](#g-03-order-of-initial-checks-in-a-function-can-be-optimized)
- [[G-04] Stack variable is only used once](#g-04-stack-variable-is-only-used-once)
- [[G-05] Consider caching `FixedPoint96.Q96 * (10 ** 18)`](#g-05-consider-caching-fixedpoint96q96--10--18)
- [[G-06] Use hardcoded address instead of `address(this)`](#g-06-use-hardcoded-address-instead-of-addressthis)
- [[G-07] Use assembly to perform efficient back-to-back calls](#g-07-use-assembly-to-perform-efficient-back-to-back-calls)
- [[G-08] Inverting the condition of an `if`-`else`-statement wastes gas](#g-08-inverting-the-condition-of-an-if-else-statement-wastes-gas)
- [[G-09] Declare variables outside of loops](#g-09-declare-variables-outside-of-loops)
- [[G-10] Do not calculate constants](#g-10-do-not-calculate-constants)
- [[G-11] Counting down in `for` statements is more gas efficient](#g-11-counting-down-in-for-statements-is-more-gas-efficient)
- [[G-12] Use assembly scratch space to build calldata for event emits](#g-12-use-assembly-scratch-space-to-build-calldata-for-event-emits)

## [G-01] Replace `type(uint<n>).max` with constants

**Saved gas per instance: 4**

```solidity
File: src/ManagedWallet.sol

37: 		activeTimelock = type(uint256).max;

68: 			activeTimelock = type(uint256).max; // effectively never

86: 		activeTimelock = type(uint256).max;
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L37-L37
- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L68-L68
- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L86-L86

##### Optimized Code:

```diff
                // Write a value so subsequent writes take less gas
-               activeTimelock = type(uint256).max;
+               activeTimelock = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
         }


        else
-                       activeTimelock = type(uint256).max; // effectively never
+                       activeTimelock = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF; // effectively never
         }


                // Reset
-               activeTimelock = type(uint256).max;
+               activeTimelock = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
                }
```

<details>
<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L91-L91
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L90-L92
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L165-L165
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L14-L14
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L50-L50
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L41-L42
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol/#L71-L73

</details>

---

## [G-02] State variables can be packed into fewer storage slots by truncating timestamp bytes

By using a `uint32` rather than a larger type for variables that track timestamps, one can save gas by using fewer storage slots per struct, at the expense of the protocol breaking after the year 2106 (when `uint32` wraps). If this is an acceptable tradeoff, if variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper

**Saved gas: 20,000**

```solidity
File: src/price_feed/PriceAggregator.sol

13: contract PriceAggregator is IPriceAggregator, Ownable
14:     {
15:     event PriceFeedSet(uint256 indexed priceFeedNum, IPriceFeed indexed newPriceFeed);
16:     event MaximumPriceFeedPercentDifferenceChanged(uint256 newMaxDifference);
17:     event SetPriceFeedCooldownChanged(uint256 newCooldown);
18:
19: 	IPriceFeed public priceFeed1; // CoreUniswapFeed by default
20: 	IPriceFeed public priceFeed2; // CoreChainlinkFeed by default
21: 	IPriceFeed public priceFeed3; // CoreSaltyFeed by default
22:
23: 	// The next time at which setPriceFeed can be called
24: 	uint256 public priceFeedModificationCooldownExpiration;
25:
26: 	// The maximum percent difference between two non-zero PriceFeed prices when aggregating price.
27: 	// When the two closest PriceFeeds (out of the three) have prices further apart than this the aggregated price is considered invalid.
28: 	// Range: 1% to 7% with an adjustment of .50%
29: 	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier
30:
31: 	// The required cooldown between calls to setPriceFeed.
32: 	// Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
33: 	// Range: 30 to 45 days with an adjustment of 5 days
34: 	uint256 public priceFeedModificationCooldown = 35 days;
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol/#L13-L34

##### Optimized Code:

```diff

        // The next time at which setPriceFeed can be called
-       uint256 public priceFeedModificationCooldownExpiration;
+       uint32 public priceFeedModificationCooldownExpiration;
+
+       // The required cooldown between calls to setPriceFeed.
+       // Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
+       // Range: 30 to 45 days with an adjustment of 5 days
+       uint32 public priceFeedModificationCooldown = 35 days;

        // The maximum percent difference between two non-zero PriceFeed prices when aggregating price.
        // When the two closest PriceFeeds (out of the three) have prices further apart than this the aggregated price is considered invalid.
        // Range: 1% to 7% with an adjustment of .50%
        uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

-       // The required cooldown between calls to setPriceFeed.
-       // Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
-       // Range: 30 to 45 days with an adjustment of 5 days
-       uint256 public priceFeedModificationCooldown = 35 days;
```

---

## [G-03] Order of initial checks in a function can be optimized

Some of the checks below involve expensive operations, such as `SLOAD`. We can reorder them so cheaper checks are done first, and expensive checks are done last. This way, if a cheaper check fails, we can avoid expensive operations.

### ManagedWallet.sol:

```solidity
42: 	function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
43: 		{
44: 		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" ); // @audit expensive operation
45: 		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
46: 		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
47:
48: 		// Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
49: 		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." ); // @audit expensive operation
```

##### Optimized Code:

```diff
        // Make a request to change the main and confirmation wallets.
        function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
                {
-               require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
                require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
                require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
+               require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );

                // Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
                require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L42-L49

#### BootstrapBallot.sol

```solidity
69: 	function finalizeBallot() external nonReentrant
70: 		{
71: 		require( ! ballotFinalized, "Ballot has already been finalized" ); // @audit expensive operation
72: 		require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
```

##### Optimized Code:

```dif
        // Ensures that the completionTimestamp has been reached and then calls InitialDistribution.distributionApproved and DAO.initialGeoExclusion if the voters have approved the ballot.
        function finalizeBallot() external nonReentrant
                {
-               require( ! ballotFinalized, "Ballot has already been finalized" );
                require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
+               require( ! ballotFinalized, "Ballot has already been finalized" );

```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol/#L69-L72

#### Pools.sol

```solidity
140: 	function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
141: 		{
142: 		require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" ); // @audit expensive operation
143: 		require( exchangeIsLive, "The exchange is not yet live" ); // @audit expensive operation
144: 		require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );
145:
146: 		require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
147: 		require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );

170: 	function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
171: 		{
172: 		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" ); // @audit expensive operation
173: 		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

219:     function withdraw( IERC20 token, uint256 amount ) external nonReentrant
220:     	{
221:     	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" ); // @audit expensive operation
222:         require( amount > PoolUtils.DUST, "Withdraw amount too small");
```

##### Optimized Code:

```diff
        function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
                {
-               require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
-               require( exchangeIsLive, "The exchange is not yet live" );
                require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );

                require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
                require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );

+               require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
+               require( exchangeIsLive, "The exchange is not yet live" );
+

        function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
                {
-               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
                require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
+               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );


     function withdraw( IERC20 token, uint256 amount ) external nonReentrant
        {
-       require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
         require( amount > PoolUtils.DUST, "Withdraw amount too small");
+       require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L140-L147
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L170-L173
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L219-L222

#### SaltRewards.sol

```solidity
117: 	function performUpkeep( bytes32[] calldata poolIDs, uint256[] calldata profitsForPools ) external
118: 		{
119: 		require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" ); // @audit expensive operation
120: 		require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
```

##### Optimized Code:

```diff
 function performUpkeep( bytes32[] calldata poolIDs, uint256[] calldata profitsForPools ) external
                {
-               require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
                require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
+               require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L117-L120

#### CollateralAndLiquidity.sol

```solidity
115:      function repayUSDS( uint256 amountRepaid ) external nonReentrant
116: 		{
117: 		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" ); // @audit expensive operation
118: 		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" ); // @audit expensive operation
119: 		require( amountRepaid > 0, "Cannot repay zero amount" );
```

##### Optimized Code:

```diff
      function repayUSDS( uint256 amountRepaid ) external nonReentrant
                {
+               require( amountRepaid > 0, "Cannot repay zero amount" );
                require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
                require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
-               require( amountRepaid > 0, "Cannot repay zero amount" );

```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol/#L115-L119

#### USDS.sol

```solidity
40: 	function mintTo( address wallet, uint256 amount ) external
41: 		{
42: 		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" ); // @audit expensive operation
43: 		require( amount > 0, "Cannot mint zero USDS" );
```

##### Optimized Code:

```diff
 function mintTo( address wallet, uint256 amount ) external
                {
-               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
                require( amount > 0, "Cannot mint zero USDS" );
+               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );

```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol/#L40-L43

#### StakingRewards.osl

```solidity
57: 	function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
58: 		{
59: 		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" ); // @audit expensive operation
60: 		require( increaseShareAmount != 0, "Cannot increase zero share" );
```

##### Optimized Code:

```diff
function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
                {
-               require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
                require( increaseShareAmount != 0, "Cannot increase zero share" );
+               require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L57-L60

---

## [G-04] Stack variable is only used once

If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend.

**Saved gas per instances: 3**

```solidity
File: src/AccessManager.sol

51:    function _verifyAccess(address wallet, bytes memory signature ) internal view returns (bool)
52:    	{
53:		bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
54:
55:		return SigningTools._verifySignature(messageHash, signature);
56:    	}
```

##### Optimized Code:

```diff
     function _verifyAccess(address wallet, bytes memory signature ) internal view returns (bool)
        {
-               bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
-
-               return SigningTools._verifySignature(messageHash, signature);
+               return SigningTools._verifySignature(keccak256(abi.encodePacked(block.chainid, geoVersion, wallet)), signature);
        }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol/#L53

<details>

<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol/#L15-L17
- https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol/#L26

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L119
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L152
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L165
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L178
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L186
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L196
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L198
  https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L233
  https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L287

- https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol/#L86

- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L223
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L246
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L250
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L257-L258
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L345
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L348
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L364
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L369

- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L89
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L94
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L105
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L146
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L157
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L171
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L189
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L201-L202
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L207
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L217
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L226
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L235
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L244
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L253
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L334
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L346
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L419

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol/#L53

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L68

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol/#L192-L193

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L40
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L63

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L356
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L399

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol/#L146
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol/#L150

</details>

## [G-05] Consider caching `FixedPoint96.Q96 * (10 ** 18)`

The following computation is repeated in multiple places in one function and can be cached to save gas:

```solidity
📁 File: src/price_feed/CoreUniswapFeed.sol

50:     function _getUniswapTwapWei( IUniswapV3Pool pool, uint256 twapInterval ) public view returns (uint256)
51:     	{
52: 		uint32[] memory secondsAgo = new uint32[](2);
53: 		secondsAgo[0] = uint32(twapInterval); // from (before)
54: 		secondsAgo[1] = 0; // to (now)
55:
56:         // Get the historical tick data using the observe() function
57:          (int56[] memory tickCumulatives, ) = pool.observe(secondsAgo);
58:
59: 		int24 tick = int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)));
60: 		uint160 sqrtPriceX96 = TickMath.getSqrtRatioAtTick( tick );
61: 		uint256 p = FullMath.mulDiv(sqrtPriceX96, sqrtPriceX96, FixedPoint96.Q96 );
62:
63: 		uint8 decimals0 = ( ERC20( pool.token0() ) ).decimals();
64: 		uint8 decimals1 = ( ERC20( pool.token1() ) ).decimals();
65:
66: 		if ( decimals1 > decimals0 )
67: 			return FullMath.mulDiv( 10 ** ( 18 + decimals1 - decimals0 ), FixedPoint96.Q96, p );
68:
69: 		if ( decimals0 > decimals1 )
70: 			return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
71:
72: 		return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
73:     	}
```

##### Optimized Code:

```diff
if ( decimals1 > decimals0 )
                        return FullMath.mulDiv( 10 ** ( 18 + decimals1 - decimals0 ), FixedPoint96.Q96, p );

+               uint _temp = FixedPoint96.Q96 * ( 10 ** 18 );
                if ( decimals0 > decimals1 )
-                       return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
+                       return ( _temp ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );

-               return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
+               return ( _temp ) / p;
        }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol/#L50-L73

## [G-06] Use hardcoded address instead of `address(this)`

It can be more gas-efficient to use a hardcoded address instead of the `address(this)` expression, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using `address(this)` requires an additional `EXTCODESIZE` operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

<details>
<summary><b>Instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol/#L27-L28

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L97
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L147
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L160
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L173

- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L170
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L246
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L300
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L307
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L365

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol/#L63

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L53

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L161-L162
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L212
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L385
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol/#L397

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L76

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L113
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L123

- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol/#L107
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol/#L139-L141
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol/#L144

- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol/#L55-L56

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol/#L88-L89

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol/#L50

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L204

</details>

## [G-07] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (`scratch space` + `free memory pointer` + `zero slot`), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

```solidity
File: src/rewards/SaltRewards.sol

41: 		salt.approve( address(stakingRewardsEmitter), type(uint256).max );
42: 		salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
```

##### Optimized Code:

```diff
-               salt.approve( address(stakingRewardsEmitter), type(uint256).max );
-               salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
+               ISalt _salt = salt;
+               assembly {
+                       // function selectors for `approve()`
+                       mstore(0x00, 0x095ea7b3)
+
+                       // store memory pointer
+                       let memptr := mload(0x40)
+                       mstore(0x20, _stakingRewardsEmitter)
+                       mstore(0x40, 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF)
+                       if iszero(call(gas(), _salt, 0x00, 0x1c, 0x44, 0x00, 0x00)) { revert(0,0) }
+
+                       mstore(0x20, _liquidityRewardsEmitter)
+                       if iszero(call(gas(), _salt, 0x00, 0x1c, 0x44, 0x00, 0x00)) { revert(0,0) }
+
+                       // restore memory pointer
+                       mstore(0x40, memptr)
+               }
        }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L41-L42

<details>
<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L56
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L59
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L62
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol/#L65

</details>

## [G-08] Inverting the condition of an `if`-`else`-statement wastes gas

Flipping the `true` and `false` blocks instead saves **_[3 gas](https://gist.github.com/IllIllI000/44da6fbe9d12b9ab21af82f14add56b9)_**.

```solidity
File: src/price_feed/CoreUniswapFeed.sol

125:         if ( ! weth_usdcFlipped )
126:         	return 10**36 / uniswapWETH_USDC;
127:         else
128:         	return uniswapWETH_USDC;
```

##### Optimized Code:

```diff
-        if ( ! weth_usdcFlipped )
-               return 10**36 / uniswapWETH_USDC;
-        else
+        if ( weth_usdcFlipped )
                return uniswapWETH_USDC;
+        else
+               return 10**36 / uniswapWETH_USDC;
                }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol/#L125-L128

## [G-09] Declare variables outside of loops

Variables should be declared outside of loops, and get overriden with each iteration of loop, By doing so we save gas cost for memory variable declaration in each iteration.

**Saved gas per instance: 15**

```solidity
File: src/arbitrage/ArbitrageSearch.sol

117: 				uint256 midpoint = (leftPoint + rightPoint) >> 1;
```

##### Optimized Code:

```diff
                        uint256 leftPoint = swapAmountInValueInETH >> 7;
                        uint256 rightPoint = swapAmountInValueInETH + (swapAmountInValueInETH >> 2); // 100% + 25% of swapAmountInValueInETH

+                       uint256 midpoint;
                        // Cost is about 492 gas per loop iteration
                        for( uint256 i = 0; i < 8; i++ )
                                {
-                               uint256 midpoint = (leftPoint + rightPoint) >> 1;
+                               midpoint = (leftPoint + rightPoint) >> 1;

                                // Right of midpoint is more profitable?
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol/#L117

<details>
<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol/#L425-L427

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L83
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L89-L91
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L94
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L109
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L112
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L115

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L62
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L65
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L118
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L121

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L66-L67

- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol/#L323
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol/#L326
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol/#L329

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L154
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L156
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L187
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L189
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L192
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L298

</details>

## [G-10] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
File: src/Salt.sol

13: 	uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;
```

##### Optimized Code:

```diff
-       uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;
+       uint256 public constant INITIAL_SUPPLY = 1e27; // 100 milion ether
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol/#L13

## [G-11] Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable. [More info](https://solodit.xyz/issues/g-02-counting-down-in-for-statements-is-more-gas-efficient-code4rena-pooltogether-pooltogether-git)

```solidity
File: src/pools/PoolStats.sol

53: 		for( uint256 i = 0; i < poolIDs.length; i++ )
```

##### Optimized Code:

```diff
-               for( uint256 i = 0; i < poolIDs.length; i++ )
+               uint256 len = poolIDs.length - 1;
+               for( uint256 i = len; i >= 0; i--)
                        _arbitrageProfits[ poolIDs[i] ] = 0;
                }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L53

<details>
<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L65
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L81
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol/#L106

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L60
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L116
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol/#L146

- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L64
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L87
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol/#L129

- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L152
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L185
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L216
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L226
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L260
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L277
  https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol/#L296

</details>

## [G-12] Use assembly scratch space to build calldata for event emits

Utilizing Solidity's assembly scratch space to build calldata for emitting events with just one or two arguments can optimize gas usage. The scratch space, a designated area in the first 64 bytes of memory, is ideal for temporary storage during assembly-level operations. By directly writing the event arguments into this area, developers can bypass the standard memory allocation process required for event emission. This approach results in gas savings, particularly for contracts where events are frequently emitted. However, such low-level optimization requires a deep understanding of Ethereum Virtual Machine (EVM) mechanics and careful coding to prevent data mishandling. Rigorous testing is essential to ensure the integrity and correct functionality of the contract.

**Saved gas per instance: 220**

```solidity
📁 File: src/AccessManager.sol

67:         emit AccessGranted( msg.sender, geoVersion );
```

##### Optimized Code:

```diff
-        emit AccessGranted( msg.sender, geoVersion );
+        // emit AccessGranted( msg.sender, geoVersion );
+                               uint256 _geoVersion = geoVersion;
+                               assembly {
+            mstore(0x00, caller())
+            mstore(0x20, _geoVersion)
+            log1(
+                0x00,
+                0x40,
+                // keccak256("AccessGranted(address,uint256)")
+                0xb4c6779ceb4a20f448e76a0e11f39bd183cff9c9dbac53df6bfcc202e2eb32f1
+            )
+        }
```

- https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol/#L67

<details>
<summary><b>Other instances:</b></summary>

- https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol/#L68

- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L54
- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol/#L83

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol/#L30

- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L248
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L251
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L254
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L257
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L260
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L263
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L266
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L269
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L272
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L275
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol/#L278

- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L127
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L144
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L151
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L163
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L174
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L182
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L268
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L343
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol/#L353

</details>
