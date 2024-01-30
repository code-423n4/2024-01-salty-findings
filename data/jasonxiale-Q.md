## [L-01] `DAO.websiteURL` doesn't have default value for first 65 days at least.
File:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L70-L109
In `DAO.constructor`, `DAO.websiteURL` doesn't set a default falue. And the only way to set `DAO.websiteURL` is by calling `Proposals.proposeWebsiteUpdate`.
Since because proposal can't be create in first 45 days because of [Proposals.sol#L83](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L83), and a ballot can't be finalized within 10 days because of [Proposals.sol#L392](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L392) is [10 days](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L45).
And after the ballot is finalized, becaue the proposal is `BallotType.SET_WEBSITE_URL`, another [BallotType.CONFIRM_SET_WEBSITE_URL](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L207-L208C88) is creates, which also requires 10 days to finalize.
So the protocol won't have websiteURL for 65 days.

# [L-02] typo in `Pools.removeLiquidity`
File: https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L187
In [Pools.sol#L187](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L187), the currect check should be:
```diff
diff --git a/src/pools/Pools.sol b/src/pools/Pools.sol
index 7adb563..1fd3f3e 100644
--- a/src/pools/Pools.sol
+++ b/src/pools/Pools.sol
@@ -184,7 +184,7 @@ contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable

                // Make sure that removing liquidity doesn't drive either of the reserves below DUST.
                // This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
-        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
+        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");

                // Switch reclaimed amounts back to the order that was specified in the call arguments so they make sense to the caller
                if (flipped)
```

## [L-03] ETH will be stuck in `ManagedWallet.sol` 
File:
    https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L59-L69
`ManagedWallet.receive` uses amount of ETH to confirms or rejects wallet proposals. When ETH is recieved in the contract, there is no way to withdraw the ETH

## [L-04] `SigningTools._verifySignature` might be suffered signature malleability attack
File:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/SigningTools.sol#L11-L28
`SigningTools._verifySignature` might be suffered signature malleability attack

## [L-05] CoreChainlinkFeed.MAX_ANSWER_DELAY is too strict
File:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L12
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L44-L49
In [CoreChainlinkFeed.latestChainlinkPrice](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L28-L60), the function checkes if the `answerDelay` between two rounds is less than `CoreChainlinkFeed.MAX_ANSWER_DELAY`, if so, `0` is returns as price.
But this check is too strict, take [BTC/USD](https://etherscan.io/address/0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c#readContract) as example, for roundId 110680464442257320397, the `updatedAt` value is 1706586947, and for roundId 110680464442257320398, the `updatedAt` is 1706590607. So the delay between those two update time is **1706590607 - 1706586947 == 3660**.
In such cases, `CoreChainlinkFeed.latestChainlinkPrice` will return **0** if the tx is in front of chainlink's price update tx.
To fix the issue, we can change `CoreChainlinkFeed.MAX_ANSWER_DELAY` a little larger such as **65 mins**

## [L-06] `PriceAggregator.setInitialFeeds` lacks of setting `priceFeedModificationCooldownExpiration`
File:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L36-L44
In [PriceAggregator.setPriceFeed](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L47C11-L62), [priceFeedModificationCooldownExpiration](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L60C3-L60C42) is set at the end of the function, it seems that `PriceAggregator.setInitialFeeds` should also be set in `PriceAggregator.setInitialFeeds`

## [L-07] Chainlink's latestRoundData return stale or incorrect result 
File:
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L32-L53

On CoreChainlinkFeed.sol, `latestRoundData` lacks of check if the returned value indicates stale data
Reference: https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/94

