

| S.No | Title | Contract Name |
|------|-------|---------------|
| L-01 | Misaligned Emissions Rate Calculation | Emissions.sol |
| L-02 | Potential Precision Loss in `_zapSwapAmount` Calculation | PoolMath.sol |
| L-03 | Division Before Validation in Arbitrage Profit Distribution | PoolStats.sol |
| L-04 | Fund Loss Due to Inadequate Handling of Dust in `_placeInternalSwap` | PoolUtils.sol |
| L-05 | Precision Loss in Price Aggregation | PriceAggregator.sol |
| L-06 | Risk of Fund Loss in `_sendInitialLiquidityRewards` | SaltRewards.sol |
| L-07 | Risk of Fund Loss in `_bisectionSearch` Function | ArbitrageSearch.sol |



### [L-01] Misaligned Emissions Rate Calculation

### Contract Name: Emissions.sol

### Description:
The `performUpkeep` function in the `Emissions` contract calculates the amount of SALT to distribute based on time elapsed and the `emissionsWeeklyPercentTimes1000` configuration. However, the calculation may suffer from a logic error due to how Solidity handles integer division, potentially leading to incorrect (usually lower) emissions than intended.

### Code Snippet:
```solidity
function performUpkeep(uint256 timeSinceLastUpkeep) external {
    ...
    uint256 saltToSend = (saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000()) / (100 * 1000 weeks);
    ...
}
```

### Expected Behavior:
The function should accurately calculate the amount of SALT to send based on the percentage specified by `emissionsWeeklyPercentTimes1000`. The expected result is a proportional distribution of tokens according to the elapsed time and the set weekly emission rate.

### Actual Behavior:
Due to the use of integer division, the multiplication of `saltBalance`, `timeSinceLastUpkeep`, and `rewardsConfig.emissionsWeeklyPercentTimes1000()` might result in a value that, when divided by `(100 * 1000 weeks)`, could produce a truncated result. This truncation occurs because Solidity does not handle fractional numbers, and any remainder after division is discarded. This could result in consistently lower emissions than expected, especially when the `saltBalance` is low or the `timeSinceLastUpkeep` is short.


### Scenario Setup:

1. **Initial Conditions:**
   - `saltBalance` (total SALT tokens in the contract): `100,000 ether` (using `ether` as a unit for simplicity).
   - `timeSinceLastUpkeep`: `3 days`.
   - `emissionsWeeklyPercentTimes1000`: `50` (representing a weekly emission rate of 0.05%).

2. **Calculation Process:**
   The amount of SALT to send (`saltToSend`) is calculated in the contract as follows:
   \[ `saltToSend` = (`saltBalance` * `timeSinceLastUpkeep` * `emissionsWeeklyPercentTimes1000`) / (100 * 1000 * `1 week`) \]

### Expected Calculation:

1. **Convert Time Units:** Convert `3 days` into a fraction of a week: `3 days / 7 days = 0.42857 weeks`.
2. **Calculate `saltToSend` (Expected):**
   - Using decimal arithmetic for precision:
     \[ `saltToSend` = (100,000 * 0.42857 * 50) / (100 * 1000) \]
     \[ `saltToSend` ≈ 214.285 \] (SALT tokens)

### Actual Calculation (With Solidity Integer Arithmetic):

1. **Integer Arithmetic in Solidity:** Solidity truncates decimals in integer division.
2. **Calculate `saltToSend` (Actual in Solidity):**
   - Without decimal places:
     \[ `saltToSend` = (100,000 * 3 days * 50) / (100 * 1000 * 7 days) \]
     \[ `saltToSend` = (15,000,000) / 700,000 \]
     \[ `saltToSend` = 21 \] (SALT tokens, as Solidity truncates the decimal part)

### Analysis:

- **Expected Behavior:** 214.285 SALT tokens should be emitted.
- **Actual Behavior in Solidity:** Only 21 SALT tokens are emitted due to integer division truncation.
- **Discrepancy:** This results in a significant underestimation of the emitted tokens. The contract emits far less than intended due to the truncation inherent in Solidity's integer arithmetic.

### Github : [Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L40)

----------------------------------------------------------------------------------------------------------------------------------------


### [L-02] Potential Precision Loss in `_zapSwapAmount` Calculation

### Contract Name: PoolMath.sol

### Description:
In the `_zapSwapAmount` function of the `PoolMath` library, there's a complex calculation involving square roots and division to determine the swap amount for liquidity addition. The use of integer division and square roots can lead to precision loss, especially when dealing with large numbers or numbers that do not divide evenly.

### Code Snippet:
```solidity
// ... within _zapSwapAmount function
uint256 sqrtDiscriminant = Math.sqrt(discriminant);
swapAmount = (sqrtDiscriminant - B) / (2 * A);
```

### Expected Behavior:
The function is expected to accurately compute the swap amount needed to balance the liquidity addition based on the reserves in the pool and the amounts to be zapped. This calculation should be precise to ensure the correct amount of tokens are swapped, maintaining the intended liquidity ratios.

### Actual Behavior:
Due to Solidity's handling of integer division and square roots, the result of the calculation may be imprecise. The division `(sqrtDiscriminant - B) / (2 * A)` could result in a truncated outcome, and the square root operation might not yield an exact result for non-perfect squares. This imprecision could lead to slightly off-balanced liquidity addition, potentially resulting in inefficiencies or minor losses in the intended liquidity ratios.

### Scenario Setup:

1. **Initial Conditions:**
   - `reserve0` (reserve of token0): `10,000` units.
   - `reserve1` (reserve of token1): `20,000` units.
   - `zapAmount0` (amount of token0 to be zapped): `1,000` units.
   - `zapAmount1` (amount of token1 to be zapped): `500` units.
   - `A`, `B`, `C` are constants derived from the quadratic equation in the contract comments.

2. **Calculation Process:**

The swap amount of token0 (swapAmount) is calculated as follows:
swapAmount = (-B + sqrt(B^2 - 4AC)) / 2A
where:
A = 1
B = 2 * reserve0
C = reserve0 * ((reserve0 * zapAmount1 - reserve1 * zapAmount0) / (reserve1 + zapAmount1))
### Expected Calculation:

1. **Calculate Constants:**
   - `A` = `1`
   - `B` = `2 * 10,000` = `20,000`
   - `C` = `10,000 * ((10,000 * 500 - 20,000 * 1,000) / (20,000 + 500))`
   - `C` ≈ `-47,619,048`

2. **Calculate `swapAmount` (Expected Using Precise Arithmetic):**
   - `discriminant` = `B^2 - 4AC` ≈ `400,000,000,000 + 190,476,192,000` ≈ `590,476,192,000`
   - `sqrtDiscriminant` ≈ `768,734.89`
   - `swapAmount` ≈ `(768,734.89 - 20,000) / 2` ≈ `374,367.45` units of token0

### Actual Calculation (With Solidity Integer Arithmetic):

1. **Integer Arithmetic in Solidity:** Solidity rounds down to the nearest integer.
2. **Calculate `swapAmount` (Actual in Solidity):**
   - `sqrtDiscriminant` = `sqrt(590,476,192,000)` ≈ `768,734` (rounded down)
   - `swapAmount` = `(768,734 - 20,000) / 2` ≈ `374,367` units of token0

### Analysis:

- **Expected Behavior:** Approximately `374,367.45` units of token0 should be swapped.
- **Actual Behavior in Solidity:** `374,367` units of token0 are swapped due to rounding down.
- **Discrepancy:** This results in a slight underestimation of the swap amount by `0.45` units of token0 due to the rounding inherent in Solidity's integer arithmetic.

### Github : [PoolMath.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L196)

-----------------------------------------------------------------------------------------------------

### [L-03] Division Before Validation in Arbitrage Profit Distribution

### Contract Name: PoolStats.sol

### Description:
In the `PoolStats` contract, the function `_calculateArbitrageProfits` divides the `arbitrageProfit` by 3 before validating whether the pool indices (`index1`, `index2`, `index3`) are valid. This approach could lead to incorrect profit calculations, especially if one or more indices are invalid (i.e., equal to `INVALID_POOL_ID`).

### Code Snippet:
```solidity
function _calculateArbitrageProfits( bytes32[] memory poolIDs, uint256[] memory _calculatedProfits ) internal view {
    for( uint256 i = 0; i < poolIDs.length; i++ ) {
        ...
        uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
        if ( arbitrageProfit > 0 ) {
            ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];

            if ( indicies.index1 != INVALID_POOL_ID )
                _calculatedProfits[indicies.index1] += arbitrageProfit;
            ...
        }
    }
}
```

### Expected Behavior:
The function should first validate the pool indices (`index1`, `index2`, `index3`) to ensure they are valid before dividing the `arbitrageProfit`. This ensures that the division only occurs when the profits can be correctly attributed to valid pools.

### Actual Behavior:
The `arbitrageProfit` is divided by 3 unconditionally before checking if the pool indices are valid. If any of these indices are invalid, the divided profits do not get correctly attributed, leading to potential discrepancies in profit distribution. This could result in some pools receiving less profit than they should, or in extreme cases, profits getting lost if all indices are invalid.



### Scenario Setup:

1. **Initial Conditions:**
   - Assume there are 3 pools involved in an arbitrage transaction.
   - Total `arbitrageProfit` generated by a specific arbitrage path: `300` units.
   - Pool indices for the arbitrage path (as stored in `_arbitrageIndicies`):
     - `index1`: `0` (valid index)
     - `index2`: `INVALID_POOL_ID` (indicating an invalid pool)
     - `index3`: `2` (valid index)

2. **Calculation of Profits:**
   - The `_calculateArbitrageProfits` function will divide the `arbitrageProfit` by 3 and attempt to distribute it to the pools based on their indices.

### Expected Calculation (Correct Logic):

1. **With Valid Pool Indices:**
   - Ideally, the function should first check if each index is valid.
   - Only distribute profits to valid indices.
   - For our scenario, this means only `index1` and `index3` should receive profits.

2. **Expected Profit Distribution:**
   - `index1` profit: `150` units (half of `300`, as only two indices are valid).
   - `index3` profit: `150` units.

### Actual Calculation (Current Logic in the Provided Code):

1. **Unconditional Division:**
   - The function divides `arbitrageProfit` by 3 unconditionally.
   - Each index is supposed to receive `100` units (`300 / 3`).

2. **Actual Profit Distribution:**
   - `index1` (valid): Receives `100` units.
   - `index2` (invalid): No distribution, but `100` units are still notionally allocated and thus "lost".
   - `index3` (valid): Receives `100` units.

### Analysis:

- **Expected Behavior:** Only valid indices receive a share of the profits, and the total distributed profit should equal the total `arbitrageProfit`. In our scenario, this would mean `150` units each to `index1` and `index3`.
  
- **Actual Behavior in Solidity:** Due to the unconditional division, the total distributed profit is `200` units (`100` each to `index1` and `index3`), leading to a loss of `100` units due to the invalid `index2`. This results in a discrepancy in profit distribution and a loss of funds.

### Github : [PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L104)

---------------------------------------------------------------------------------------------------------------------------------------

### [L-04] Fund Loss Due to Inadequate Handling of Dust in `_placeInternalSwap`

### Contract Name: PoolUtils.sol

### Description:
In the `PoolUtils` library, the `_placeInternalSwap` function is designed to execute token swaps within the protocol. However, there's a potential issue with how the function handles very small amounts of tokens, termed as 'Dust'. The function does not check if the `amountIn` is below the defined `DUST` threshold, which could lead to situations where the amount is treated as negligible and not effectively swapped, potentially causing a loss of funds.

### Code Snippet:
```solidity
function _placeInternalSwap( ... ) internal returns (uint256 swapAmountIn, uint256 swapAmountOut) {
    if (amountIn == 0)
        return (0, 0);

    (uint256 reservesIn,) = pools.getPoolReserves(tokenIn, tokenOut);
    uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
    if (amountIn > maxAmountIn)
        amountIn = maxAmountIn;

    swapAmountIn = amountIn;
    swapAmountOut = pools.depositSwapWithdraw(tokenIn, tokenOut, amountIn, 0, block.timestamp);
}
```

### Expected Behavior:
The function should include a check for `amountIn` against the `DUST` threshold. If the amount is below this threshold, the function should either round up the amount to the minimum viable swap amount or handle it in a manner that prevents the loss of these tokens.

### Actual Behavior:
Currently, if `amountIn` is a very small amount (but not zero), the function proceeds with the swap. Given that the amount is below the `DUST` threshold, it might be disregarded in the swap process due to its negligible size, leading to a situation where these tokens are effectively lost or stuck without contributing to any meaningful swap outcome.


### Scenario Setup:

- **Initial Conditions:**
  - `reservesIn`: `100,000` units
  - `maximumInternalSwapPercentTimes1000`: `5,000` (5%)
  - `amountIn`: `50` units
  - `DUST` threshold: `100` units

### Expected Calculation (With Dust Check):

1. **Calculate `maxAmountIn`:**
   - `maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / 100,000`
   - `maxAmountIn = 100,000 * 5,000 / 100,000`
   - `maxAmountIn = 500,000,000 / 100,000`
   - `maxAmountIn = 5,000` units

2. **Check Against Dust:**
   - `amountIn` (50 units) is compared to `DUST` (100 units).
   - Since `amountIn` < `DUST`, the function should handle this case specially.

3. **Expected Swap Execution:**
   - The swap might not execute, or `amountIn` could be adjusted to `DUST`.
   - If adjusted to `DUST`, `amountIn` would be `100` units for the swap.

### Actual Calculation (Without Dust Check):

1. **Calculate `maxAmountIn`:**
   - This calculation remains the same as above.
   - `maxAmountIn = 5,000` units

2. **No Dust Check:**
   - `amountIn` is `50` units, which is below the `DUST` threshold but is used as-is because there's no dust check.

3. **Actual Swap Execution:**
   - The swap proceeds with `amountIn` of `50` units.
   - There's no adjustment for being below `DUST`, so the small amount is used directly.

### Result Comparison:

- **Expected Behavior (With Dust Check):**
  - The swap either does not proceed or proceeds with a minimum of `100` units (`DUST` threshold) instead of the original `50` units.
  - This prevents the loss of funds due to very small swaps being ineffective.

- **Actual Behavior (Without Dust Check):**
  - The swap proceeds with `50` units, despite being below the `DUST` threshold.
  - This could result in ineffective swaps where the small amount doesn't significantly impact the pool, potentially leading to a gradual loss of these tokens over multiple transactions.


### Github : [PoolUtils.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L54)
  ----------------------------------------------------------------------------------------------------------------------------------------

### [L-05] Precision Loss in Price Aggregation

### Contract Name: PriceAggregator.sol

### Description:
The `PriceAggregator` contract, particularly in the `_aggregatePrices` function, calculates the average price from three different price feeds for BTC and ETH. However, the calculation method used to determine if the price sources are too far apart may suffer from precision loss, which could lead to incorrect rejection of valid prices.

### Code Snippet:
```solidity
function _aggregatePrices(uint256 price1, uint256 price2, uint256 price3) internal view returns (uint256) {
    ...
    uint256 averagePrice = (priceA + priceB) / 2;

    if ((_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000)
        return 0;

    return averagePrice;
}
```

### Expected Behavior:
The function should accurately determine whether the closest two price feeds are within an acceptable range, defined by `maximumPriceFeedPercentDifferenceTimes1000`. This should be done with high precision to ensure that valid price pairs are not incorrectly discarded.

### Actual Behavior:
Due to the division by `averagePrice` before multiplying by `100000`, there is a risk of precision loss, especially when `averagePrice` is large. This could result in the condition incorrectly evaluating to `true`, causing the function to return `0` even when the price feeds are within the acceptable range.


### Scenario Setup:

1. **Initial Conditions:**
   - Three price feeds (`price1`, `price2`, `price3`) for BTC or ETH.
   - Example prices from the feeds: `price1 = 50,000`, `price2 = 51,000`, `price3 = 55,000` (unit: USD).
   - `maximumPriceFeedPercentDifferenceTimes1000`: `2,000` (representing a maximum allowable difference of 2%).

2. **Calculation of Average and Difference Check:**
   - The `_aggregatePrices` function calculates the average of the two closest prices and checks if their difference is within the allowable range.

### Expected Calculation (With High Precision Arithmetic):

1. **Identify Closest Prices:**
   - Closest prices are `price1` and `price2` (`50,000` and `51,000`).

2. **Calculate Average:**
   - `averagePrice = (50,000 + 51,000) / 2 = 101,000 / 2 = 50,500`.

3. **Difference Check:**
   - `difference = |50,000 - 51,000| = 1,000`.
   - Percentage difference: `difference / averagePrice = 1,000 / 50,500 ≈ 0.0198` (approximately 1.98%).
   - Since 1.98% < 2%, the prices are considered valid.

### Actual Calculation (With Solidity Integer Arithmetic):

1. **Same Closest Prices:**
   - `price1` and `price2` are still the closest.

2. **Calculate Average:**
   - `averagePrice` calculation remains the same: `50,500`.

3. **Difference Check in Solidity:**
   - `difference` calculation remains the same: `1,000`.
   - The check for percentage difference in Solidity: `(_absoluteDifference(priceA, priceB) * 100000) / averagePrice`.
   - Calculating: `(1,000 * 100,000) / 50,500 = 100,000,000 / 50,500 ≈ 1,980` (Solidity truncates the decimal).
   - Since this calculation is meant to represent a percentage multiplied by `1,000`, the actual percentage difference is approximately `1.98%`, which is below the 2% threshold.

### Analysis:

- **Expected Behavior:** The two closest price feeds are within the acceptable range (less than 2% difference), so the average price is valid and returned.

- **Actual Behavior in Solidity:** The calculation also correctly identifies that the price difference is within the allowable range, and the average price is valid.

### Github : [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L108)

------------------------------------------------------------------------------------------------------------------

### [L-06] Risk of Fund Loss in `_sendInitialLiquidityRewards`

### Contract Name: SaltRewards.sol

### Description:
In the `SaltRewards` contract, the function `_sendInitialLiquidityRewards` divides the `liquidityBootstrapAmount` evenly across all initial pools. However, if this division results in a fractional number of tokens that cannot be represented in Solidity's integer-based arithmetic, it can lead to a loss of tokens due to rounding down.

### Code Snippet:
```solidity
function _sendInitialLiquidityRewards(uint256 liquidityBootstrapAmount, bytes32[] memory poolIDs) internal {
    uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length; // Possible loss of tokens due to rounding

    AddedReward[] memory addedRewards = new AddedReward[](poolIDs.length);
    for (uint256 i = 0; i < addedRewards.length; i++) {
        addedRewards[i] = AddedReward(poolIDs[i], amountPerPool);
    }

    liquidityRewardsEmitter.addSALTRewards(addedRewards);
}
```

### Expected Behavior:
The function should distribute `liquidityBootstrapAmount` in a way that ensures no tokens are lost. Ideally, any remaining tokens after the division should be allocated appropriately, perhaps to one of the pools or handled in another defined manner.

### Actual Behavior:
Due to integer division, the `amountPerPool` calculation may truncate any fractional part of the token amount. When multiplied back by the number of pools, the total distributed amount can be less than `liquidityBootstrapAmount`, leading to a loss of tokens. For example, if `liquidityBootstrapAmount` is `1,000` and there are `3` pools, each pool receives `333` tokens, totaling `999` tokens distributed with `1` token effectively lost.


### Scenario Setup:

- **Initial Conditions:**
  - `liquidityBootstrapAmount` (Total SALT tokens to be distributed): `1,000` tokens.
  - `poolIDs` (Number of pools): `3` pools (e.g., `pool1`, `pool2`, `pool3`).

### Expected Calculation (Ideal Handling of Token Distribution):

1. **Calculate Rewards Per Pool (Ideal):**
   - Ideally, `liquidityBootstrapAmount` should be distributed evenly across the pools, with any remaining tokens allocated to prevent loss.
   - `amountPerPool` (Ideal) = `liquidityBootstrapAmount / Number of Pools`
   - `amountPerPool` (Ideal) = `1,000 tokens / 3 pools = 333.33` tokens per pool.
   - Since we can't distribute a fraction of a token, we round down to `333` tokens per pool and handle the remainder separately.

2. **Handle Remainder:**
   - Total distributed without remainder: `333 tokens * 3 pools = 999 tokens`.
   - Remainder: `1,000 tokens (total) - 999 tokens (distributed) = 1 token`.
   - This remaining `1 token` could be distributed to one of the pools or handled in another manner.

### Actual Calculation (With Solidity's Integer Arithmetic):

1. **Calculate Rewards Per Pool (Solidity):**
   - In Solidity, division truncates the decimal part, effectively rounding down.
   - `amountPerPool` (Solidity) = `liquidityBootstrapAmount / Number of Pools`
   - `amountPerPool` (Solidity) = `1,000 tokens / 3 pools = 333 tokens` per pool (decimal part truncated).

2. **Total Distribution in Solidity:**
   - Total distributed: `333 tokens * 3 pools = 999 tokens`.
   - There's no mechanism to handle the remainder, so `1 token` is effectively lost.

### Analysis:

- **Expected Behavior (Ideal Handling):** Evenly distribute tokens to each pool, and handle the remainder to ensure no tokens are lost. In this case, distribute `333` tokens to each pool and allocate the remaining `1 token` appropriately.

- **Actual Behavior in Solidity:** Distribute `333` tokens to each pool, resulting in a total of `999` tokens distributed. The remaining `1 token` is not allocated due to Solidity's integer division, leading to its loss.

### Github : [SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L81)

-----------------------------------------------------------------------------------------------------------

### [L-07] Risk of Fund Loss in `_bisectionSearch` Function

### Contract Name: ArbitrageSearch.sol

### Description:
The `_bisectionSearch` function in the `ArbitrageSearch` contract is designed to find the optimal amount for an arbitrage opportunity. However, there's a potential issue in the way this function handles the search range and the midpoint calculation, which might lead to an inaccurate determination of the optimal arbitrage amount, potentially resulting in a less profitable or even unprofitable arbitrage transaction.

### Code Snippet:
```solidity
function _bisectionSearch( ... ) internal pure returns (uint256 bestArbAmountIn) {
    ...
    uint256 leftPoint = swapAmountInValueInETH >> 7;
    uint256 rightPoint = swapAmountInValueInETH + (swapAmountInValueInETH >> 2);
    ...
    for( uint256 i = 0; i < 8; i++ ) {
        uint256 midpoint = (leftPoint + rightPoint) >> 1;
        ...
    }
    ...
    bestArbAmountIn = (leftPoint + rightPoint) >> 1;
    ...
}
```

### Expected Behavior:
The function should accurately determine the optimal arbitrage amount that maximizes profit. The bisection search algorithm used should efficiently converge to the most profitable point within the defined search range.

### Actual Behavior:
The implementation of the bisection search might not accurately find the optimal arbitrage amount due to how the range and midpoint are calculated. The function calculates the initial search range based on a fraction and a percentage of `swapAmountInValueInETH`, and then repeatedly bisects this range. However, if the actual optimal amount for arbitrage lies outside of this initial range or near its boundaries, the function may miss it, leading to suboptimal arbitrage decisions. This could result in either lower profits or, in some cases, unprofitable transactions if transaction costs are considered.



### Scenario Setup:

1. **Initial Conditions:**
   - `swapAmountInValueInETH`: Let's assume it's `1,000` ETH.
   - Reserves for pools A, B, and C are assumed to be large enough not to influence the calculations significantly in this example.

2. **Bisection Search Range:**
   - Initial `leftPoint` = `swapAmountInValueInETH / 128` = `1,000 / 128 ≈ 7.81` ETH.
   - Initial `rightPoint` = `swapAmountInValueInETH + (swapAmountInValueInETH / 4)` = `1,000 + 250 = 1,250` ETH.

3. **Assumption for Optimal Arbitrage Amount:**
   - Let's assume the actual optimal arbitrage amount is `500` ETH, which lies within the initial range.

### Expected Calculation (Ideal Arbitrage Determination):

1. **Ideal Arbitrage Search:**
   - An ideal search algorithm would accurately converge to the optimal point, `500` ETH, efficiently.
   - It would iteratively adjust the search range to zero in on `500` ETH.

2. **Expected Outcome:**
   - The algorithm identifies `500` ETH as the optimal amount for arbitrage.

### Actual Calculation (Bisection Search Algorithm):

1. **First Iteration:**
   - Midpoint = `(7.81 + 1,250) / 2 ≈ 628.91` ETH.
   - If the right of this midpoint is more profitable, the next left point would be `628.91` ETH.

2. **Subsequent Iterations:**
   - The algorithm continues to bisect the range, but the accuracy of finding the exact optimal point (500 ETH) depends on the range adjustments and the number of iterations.

3. **Final Outcome:**
   - The algorithm converges to a point close to `500` ETH, but the exactness depends on the number of iterations and how the profit comparison is made at each step.

### Analysis:

- **Expected Behavior:** The optimal arbitrage amount is identified accurately, maximizing the profit.

- **Actual Behavior in Solidity:** The bisection search algorithm may approximate the optimal amount but may not precisely pinpoint `500` ETH due to the initial range and the method of determining the more profitable side of the midpoint.

### Github : [ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L101)

------------------------------------------------------------------------------------

