## 1 Typo in SigningTools.sol comment

https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L10

Here,
// Verify that the messageHash was signed by the `authoratative` signer.
should be,
// Verify that the messageHash was signed by the `authoritative` signer.

## 2 Typo in PriceAggregator.sol comments

// Allows time to evaluate the performance of the recently `updatef` PriceFeed before further updates are made.
should be,
// Allows time to evaluate the performance of the recently `updated` PriceFeed before further updates are made.
instead.


https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L32

## 3 Typo in PoolStats.sol `updateArbitrageIndicies()` comments

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L75-#L76
// Traverse the current whitelisted poolIDs and update the `indicies` of each pool that would contribute to arbitrage for it.
	// Maps poolID(arbToken2, arbToken3) => the `indicies` (within the whitelistedPools array) of the pools involved in WETH->arbToken2->arbToken3->WETH arbitrage.

Should be,
// Traverse the current whitelisted poolIDs and update the `indices` of each pool that would contribute to arbitrage for it.
	// Maps poolID(arbToken2, arbToken3) => the `indices` (within the whitelistedPools array) of the pools involved in WETH->arbToken2->arbToken3->WETH arbitrage
If you mean the plural of index
I understand that you chose to name the `updateArbitrageIndicies()` function with the `i`, but you are free to do that since it is a function name, but it is a 'no, no' in comments.