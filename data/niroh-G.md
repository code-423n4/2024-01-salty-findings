# Gas Optimizations

## [G-01] Arbitrage Function can save gas by returning the expected arb profit of each iteration and exiting if the last incremental improvement is below some threshold 
### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L63
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L120
### Description
1. The _bisectionSearch function in ArbitrageSearch searches for the optimal input amount in terms of arbitrage profit. It iterates 8 times, calling _rightMoreProfitable each time to get an indication of weather it should iterate to the left or right half of the current range.
2. _rightMoreProfitable calculates the arb profit at two adgacent points to produce an answer.
3. In many cases, the iteration reaches very small incremental improvements after a small number of iterations (i.e. when trades or profit range are small).
4. Running the full 8 iterations has a significant effect on swap gas costs which overweighs miniscule improvements of arbitrage profits
5. Since _rightMoreProfitable calculates the profit at the current point anyway, the current profit can be returned to _bisectionSearch, and an exit condition added to the loop that exits if the last incremental improvement is smaller than a certain threshold (i.e. 0.000001 WETH or 0.2 cents at current prices)


## [G-02] Dao members poolsConfig,  stakingConfig,  rewardsConfig,  stableConfig,  daoConfig and priceAggregator should be defined in the Parameters parent contract, saving an unnessecary transfer of all of them with each call to _executeParameterChange. 

### Github Links
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Parameters.sol#L57

### Description
Since the Parameters contract is only used as a parent contract for Dao it would save gas to define the state variables mentioned above in Parameters. This would save the added gas cost of pushing these 6 addresses to the stack (roughly 60x6 = 360 gas units) with each call to _executeParameterChange.







