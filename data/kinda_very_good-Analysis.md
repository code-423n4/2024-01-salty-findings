

### [H-1] The first user to deposit could set a bad ratio 

**Description:** Depositing and collateral addition does not rely on price returned by aggregator meaning the first depositer would determine the price ratio of the pool

**Impact:** The protocol would be griefed 



**Recommended Mitigation:** The initial addition of liquidity especially in the important pools (weth,wbtc and dai) should be made off a price feed 



### [H-2] setInitialFeeds does not use cool down 

**Description:** PriceAggregator::setInitialFeeds allows the owner to set the priceFeeds before the cooldown period is over

**Impact:** owner could set bad feed 



**Recommended Mitigation:** PriceAggregator::setInitialFeeds should only be callable once 

### [L-1] Aggregate price could revert on correct price 

**Description:** AggregatePrice::_aggregate would return zero if the two closest prices are exactly the same

**Impact:** Users could be griefed 


```
if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )

if the two prices are exactly the same their absolute difference would be zero 

```

**Recommended Mitigation:** A check should be included to check if the two prices are exactly the same and then one of the prices could be returned 


### [I-1] Named mappings should be used 

### Time spent:
22 hours