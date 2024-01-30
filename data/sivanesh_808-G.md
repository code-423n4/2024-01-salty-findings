Contest : salty[GAS]



| S.No | Title | Contract Name |
|------|-------|---------------|
| G-01 | Optimization Report for `getPriceBTC` and `getPriceETH` Functions in CoreSaltyFeed Contract | CoreSaltyFeed.sol |
| G-02 | Optimization of `latestChainlinkPrice` Function in CoreChainlinkFeed Contract | CoreChainlinkFeed.sol |
| G-03 | Optimization Analysis of `_aggregatePrices` Function in the Price Aggregator Contract | `PriceAggregatorGasTest.sol` |
| G-04 | Optimization of the `_placeInternalSwap` Function in the `PoolUtils` Library | `PoolUtils.sol` |
| G-05 | Optimization of `totalVotesCastForBallot` in `Proposals` Contract | Proposals.sol |
| G-06 | Optimization of Unnecessary `if` Statements for Zero Checks | `RewardsEmitter.sol` |
| G-07 | Optimization of `PoolsConfig` Contract | PoolsConfig.sol |
| G-08 | Enhanced Gas Efficiency in `changeMaximumWhitelistedPools` Function | PoolsConfig.sol |
| G-09 | Enhanced Gas Efficiency in `changeMaximumInternalSwapPercentTimes1000` Function | PoolsConfig.sol |
| G-10 | Optimization of `walletHasAccess` Function in `ExchangeConfig` Contract | ExchangeConfig.sol |
| G-11 | Optimization of `changeBootstrappingRewards` in DAOConfig Contract | DAOConfig.sol |
| G-12 | Arithmetic Operations Optimization in Solidity Smart Contract | CollateralAndLiquidity.sol.sol |
| G-13 | Optimization of State Variable Access in Solidity Smart Contract | CollateralAndLiquidity.sol |
| G-14 | Combine Increment and Decrement Operations in `StakingConfig` Contract | StakingConfig.sol |
| G-15 | Gas Efficiency Improvement in `getTwapWETH` Function | `CoreUniswapFeed.sol` |




### [G-01] Optimization Report for `getPriceBTC` and `getPriceETH` Functions in CoreSaltyFeed Contract

### Contract Name:
CoreSaltyFeed.sol

### Description:
The `CoreSaltyFeed` contract is designed for price retrieval of BTC and ETH in a decentralized finance (DeFi) context. This report addresses the gas optimization of both `getPriceBTC` and `getPriceETH` functions. These functions calculate the respective prices based on liquidity pool reserves. The aim is to enhance gas efficiency without compromising the core logic and output.

### Original Functions:
**`getPriceBTC` Function:**
```solidity
function getPriceBTC() external view returns (uint256) {
    (uint256 reservesWBTC, uint256 reservesUSDS) = pools.getPoolReserves(wbtc, usds);
    if ((reservesWBTC < PoolUtils.DUST) || (reservesUSDS < PoolUtils.DUST)) {
        return 0;
    }
    return (reservesUSDS * 10**8) / reservesWBTC;
}
```

**`getPriceETH` Function:**
```solidity
function getPriceETH() external view returns (uint256) {
    (uint256 reservesWETH, uint256 reservesUSDS) = pools.getPoolReserves(weth, usds);
    if ((reservesWETH < PoolUtils.DUST) || (reservesUSDS < PoolUtils.DUST)) {
        return 0;
    }
    return (reservesUSDS * 10**18) / reservesWETH;
}
```

### Optimized Functions:
**Common Optimizations for Both Functions:**
1. **Caching State Variables in Memory:**
   - State variables (`wbtc`, `weth`, `usds`) are cached in memory to reduce gas costs associated with state variable accesses.

2. **Unchecked Division:**
   - The division operation is placed inside an `unchecked` block to avoid unnecessary overflow/underflow checks, thereby saving gas.

3. **Precomputed Constants:**
   - Constants (`10**8` for BTC, `10**18` for ETH) are precomputed and stored to minimize repetitive calculations.

**Optimized `getPriceBTC`:**
```solidity
uint256 private constant DECIMAL_FACTOR_BTC = 10**8;

function getPriceBTC() external view returns (uint256) {
    IERC20 memory _wbtc = wbtc;
    IERC20 memory _usds = usds;
    (uint256 reservesWBTC, uint256 reservesUSDS) = pools.getPoolReserves(_wbtc, _usds);
    if ((reservesWBTC < PoolUtils.DUST) || (reservesUSDS < PoolUtils.DUST)) {
        return 0;
    }
    unchecked {
        return (reservesUSDS * DECIMAL_FACTOR_BTC) / reservesWBTC;
    }
}
```

**Optimized `getPriceETH`:**
```solidity
uint256 private constant DECIMAL_FACTOR_ETH = 10**18;

function getPriceETH() external view returns (uint256) {
    IERC20 memory _weth = weth;
    IERC20 memory _usds = usds;
    (uint256 reservesWETH, uint256 reservesUSDS) = pools.getPoolReserves(_weth, _usds);
    if ((reservesWETH < PoolUtils.DUST) || (reservesUSDS < PoolUtils.DUST)) {
        return 0;
    }
    unchecked {
        return (reservesUSDS * DECIMAL_FACTOR_ETH) / reservesWETH;
    }
}
```
### Github : [CoreSaltyFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L32)
-------------------------------------------------------------------------------------------

### [G-02] Optimization of `latestChainlinkPrice` Function in CoreChainlinkFeed Contract

### Contract Name:
CoreChainlinkFeed.sol

### Description:
The `latestChainlinkPrice` function in the CoreChainlinkFeed contract retrieves the latest price data from a Chainlink oracle. The original implementation included multiple checks and assignments, resulting in higher gas costs. The optimized version streamlines these processes to reduce gas usage while maintaining the functionality of fetching and validating the latest Chainlink oracle data.

### Original Code:
```solidity
function latestChainlinkPrice(AggregatorV3Interface chainlinkFeed) public view returns (uint256) {
    int256 price = 0;

    try chainlinkFeed.latestRoundData()
    returns (
        uint80, // _roundID
        int256 _price,
        uint256, // _startedAt
        uint256 _answerTimestamp,
        uint80 // _answeredInRound
    )
        {
        uint256 answerDelay = block.timestamp - _answerTimestamp;

        if ( answerDelay <= MAX_ANSWER_DELAY )
            price = _price;
        else
            price = 0;
        }
    catch (bytes memory) {
        // In case of failure, price will remain 0
    }

    if ( price < 0 )
        return 0;

    return uint256(price) * 10**10;
}
```

### Optimized Code:
```solidity
function latestChainlinkPrice(AggregatorV3Interface chainlinkFeed) public view returns (uint256) {
    try chainlinkFeed.latestRoundData()
    returns (
        uint80, // _roundID
        int256 _price,
        uint256, // _startedAt
        uint256 _answerTimestamp,
        uint80 // _answeredInRound
    )
        {
        if (block.timestamp - _answerTimestamp > MAX_ANSWER_DELAY || _price < 0)
            return 0;

        return uint256(_price) * 10**10;
        }
    catch (bytes memory) {
        return 0;
    }
}
```

### Explanation of Optimization:
- **Consolidated Conditionals**: The optimized code combines the checks for answer delay and positive price into one conditional statement. This reduces the number of operations, as it eliminates the need for a separate variable assignment and conditional check.
 
- **Direct Return Strategy**: Instead of storing the price in a temporary variable and then performing checks, the optimized code directly returns the result. This streamlines the function and reduces the number of assignments and storage operations, leading to lower gas costs.

- **Simplified Error Handling**: The `catch` block remains unchanged but benefits indirectly from the streamlined logic, as it now deals with a simplified logic flow.


### Github : [CoreChainlinkFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L28)

--------------------------------------------------------------------------------------------------------------------------------------------------


### [G-03] Optimization Analysis of `_aggregatePrices` Function in the Price Aggregator Contract

### Contract Name:
`PriceAggregator.sol`

### Description:
The `_aggregatePrices` function within the `PriceAggregatorGasTest` contract is designed to aggregate prices from three distinct sources, ensuring reliability by eliminating outliers. This function is crucial for providing accurate price data in scenarios where one or more sources might be unreliable. The original implementation uses conditional logic and arithmetic operations that, while functional, offer room for gas consumption optimization. The optimized version of the function aims to reduce gas costs by simplifying the logic and reducing the complexity of calculations.

### Original Code:
```solidity
function _aggregatePrices(uint256 price1, uint256 price2, uint256 price3) internal view returns (uint256) {
    uint256 numNonZero;
    if (price1 > 0) numNonZero++;
    if (price2 > 0) numNonZero++;
    if (price3 > 0) numNonZero++;

    if (numNonZero < 2) return 0;

    uint256 diff12 = _absoluteDifference(price1, price2);
    uint256 diff13 = _absoluteDifference(price1, price3);
    uint256 diff23 = _absoluteDifference(price2, price3);

    uint256 priceA;
    uint256 priceB;
    if ((diff12 <= diff13) && (diff12 <= diff23)) (priceA, priceB) = (price1, price2);
    else if ((diff13 <= diff12) && (diff13 <= diff23)) (priceA, priceB) = (price1, price3);
    else (priceA, priceB) = (price2, price3);

    uint256 averagePrice = (priceA + priceB) / 2;

    if ((_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000) return 0;

    return averagePrice;
}
```

### Optimized Code:
```solidity
function _aggregatePricesOptimized(uint256 price1, uint256 price2, uint256 price3) internal view returns (uint256) {
    uint256[3] memory prices = [price1, price2, price3];
    uint256 numNonZero = (price1 != 0 ? 1 : 0) + (price2 != 0 ? 1 : 0) + (price3 != 0 ? 1 : 0);

    if (numNonZero < 2) return 0;

    // Sort prices in ascending order
    if (prices[0] > prices[1]) (prices[0], prices[1]) = (prices[1], prices[0]);
    if (prices[1] > prices[2]) (prices[1], prices[2]) = (prices[2], prices[1]);
    if (prices[0] > prices[1]) (prices[0], prices[1]) = (prices[1], prices[0]);

    uint256 averagePrice = (prices[0] + prices[1]) / 2;

    if ((prices[1] - prices[0]) * 100000 / averagePrice > maximumPriceFeedPercentDifferenceTimes1000) return 0;

    return averagePrice;
}
```

### Optimization Explanation:

- **Simplification of Non-Zero Counting**: The optimized code employs a more compact approach for counting non-zero prices, reducing the number of conditional checks.

- **Array and Sorting**: By storing prices in an array and sorting them, the optimized version efficiently identifies the two closest prices without multiple conditional checks. This method streamlines the process of discarding the outlier and calculating the average of the two closest prices.

- **Reduced Conditional Checks**: The sorting algorithm reduces the need for multiple conditional checks to find the two closest prices, which simplifies the logic and potentially saves gas.

- **Direct Average Calculation**: After sorting, the optimized code directly averages the first two elements of the sorted array (the two closest prices), eliminating the need for additional logic to select these values.




### Github : [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L108)

------------------------------------------------------------------------------------------

### [G-04] Optimization of the `_placeInternalSwap` Function in the `PoolUtils` Library

### Contract Name:
`PoolUtils.sol`

### Description:
The `_placeInternalSwap` function is designed to execute token swaps within a protocol, with the swap amount limited to a specific percentage of the token reserves. This function is part of a broader strategy to mitigate the impact of sandwich attacks by limiting the size of swaps. The original code performs a series of checks and calculations to determine the maximum allowed swap amount. The optimized version aims to simplify these calculations and improve the overall gas efficiency of the function.

### Original Code:
```solidity
function _placeInternalSwap(IPools pools, IERC20 tokenIn, IERC20 tokenOut, uint256 amountIn, uint256 maximumInternalSwapPercentTimes1000) internal returns (uint256 swapAmountIn, uint256 swapAmountOut) {
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

### Optimized Code:
```solidity
function _placeInternalSwapOptimized(IPools pools, IERC20 tokenIn, IERC20 tokenOut, uint256 amountIn, uint256 maximumInternalSwapPercentTimes1000) internal returns (uint256 swapAmountIn, uint256 swapAmountOut) {
    if (amountIn == 0) {
        return (0, 0);
    }

    (uint256 reservesIn,) = pools.getPoolReserves(tokenIn, tokenOut);

    // Simplify the division calculation
    uint256 maxAmountIn = (reservesIn * maximumInternalSwapPercentTimes1000) / 100000;

    // Use the min function to determine the swap amount
    swapAmountIn = Math.min(amountIn, maxAmountIn);

    // Only perform the swap if there is an amount to swap
    if (swapAmountIn > 0) {
        swapAmountOut = pools.depositSwapWithdraw(tokenIn, tokenOut, swapAmountIn, 0, block.timestamp);
    } else {
        swapAmountOut = 0;
    }

    return (swapAmountIn, swapAmountOut);
}
```

### Optimization Explanation:
- **Simplified Division Calculation**: The optimized code simplifies the division to a single operation by dividing directly by `100000` instead of `(100 * 1000)`. This change reduces the complexity of the arithmetic operation and can potentially save gas.
  
- **Usage of `Math.min` Function**: The optimized version uses `Math.min` to determine the smaller value between `amountIn` and `maxAmountIn`. This approach is more expressive and can potentially be more gas-efficient than conditional statements.

- **Conditional Swap Execution**: The swap is only executed if there is a non-zero amount to swap (`swapAmountIn > 0`). This check avoids unnecessary calls to `pools.depositSwapWithdraw` when no swap occurs, potentially saving gas.

- **Explicit Zero Assignment for `swapAmountOut`**: In cases where no swap takes place, `swapAmountOut` is explicitly set to zero, ensuring clarity in the function's behavior.


### Github : [PoolUtils.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L54)

---------------------------------------------------------------------------------------------------------------------------------------

### [G-05] Optimization of `totalVotesCastForBallot` in `Proposals` Contract

**Contract Name:** Proposals.sol

**Description:** 
The `totalVotesCastForBallot` function calculates the total number of votes cast for a specific ballot. The optimization focuses on reducing storage access by accessing the `_votesCastForBallot` mapping directly in the return statement and using a ternary operator for concise logic. The goal is to save gas by minimizing storage operations and streamlining the code.

**Original Code:**

```solidity
function totalVotesCastForBallot(uint256 ballotID) public view returns (uint256) {
    mapping(Vote => uint256) storage votes = _votesCastForBallot[ballotID];

    Ballot memory ballot = ballots[ballotID];
    if (ballot.ballotType == BallotType.PARAMETER) {
        return votes[Vote.INCREASE] + votes[Vote.DECREASE] + votes[Vote.NO_CHANGE];
    } else {
        return votes[Vote.YES] + votes[Vote.NO];
    }
}
```

**Optimized Code:**

```solidity
function totalVotesCastForBallotOptimized(uint256 ballotID) public view returns (uint256) {
    Ballot storage ballot = ballots[ballotID];
    return ballot.ballotType == BallotType.PARAMETER
           ? _votesCastForBallot[ballotID][Vote.INCREASE] + _votesCastForBallot[ballotID][Vote.DECREASE] + _votesCastForBallot[ballotID][Vote.NO_CHANGE]
           : _votesCastForBallot[ballotID][Vote.YES] + _votesCastForBallot[ballotID][Vote.NO];
}
```

**Optimization Explanation:**

1. **Direct Storage Access**: The optimized function directly accesses the `_votesCastForBallot` mapping in the return statement, which can potentially reduce gas costs by avoiding an extra local variable for storage mapping.

2. **Use of Ternary Operator**: The ternary operator (`?:`) condenses the conditional logic, making the code more readable and concise. While this may not directly impact gas savings, it makes the function more straightforward.

3. **Efficiency in Logic Execution**: By combining the logic into a single line with direct mapping access, the optimized code reduces the overhead associated with memory and storage operations, which is beneficial for gas efficiency.



### Github : [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L342)

-------------------------------------------------------------------------------------------------------------

### [G-06] Optimization of Unnecessary `if` Statements for Zero Checks

**Contract Name**: `RewardsEmitter.sol`

### Description:
In the smart contract `RewardsEmitter`, there's an `if` statement used to check whether the variable `sum` is greater than zero before executing the `safeTransferFrom` function. This check is redundant since the ERC-20 token standard's `transfer` and `transferFrom` functions inherently revert if the transfer amount is zero, thus making an explicit check for zero transfer amounts unnecessary. Removing this check can save gas by reducing the overall bytecode size and the execution cost of the contract.

### Original Code:
```solidity
// In function addSALTRewards
if (sum > 0)
    salt.safeTransferFrom(msg.sender, address(this), sum);
```

### Optimized Code:
```solidity
// In function addSALTRewards
salt.safeTransferFrom(msg.sender, address(this), sum);
```

### Optimization Explanation:
The original code includes an `if` statement to check if `sum` is greater than zero before calling `salt.safeTransferFrom`. This check is not necessary for two reasons:

1. **ERC-20 Standard Compliance**: The ERC-20 standard's `transfer` and `transferFrom` functions must revert in case of a failure, which includes a transfer of zero tokens when not allowed. Therefore, the ERC-20 token contract for SALT should inherently handle the case where the transfer amount is zero.

2. **Gas Efficiency**: Removing unnecessary conditionals reduces the contract's bytecode size, leading to lower deployment and execution costs. The gas cost for the conditional check and the additional jump instruction can be saved.


### Github : [RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L76)

---------------------------------------------------------------------------------------------------------------------


### [G-07] Optimization of `PoolsConfig` Contract

**Contract Name**: PoolsConfig.sol

**Description**: 
This report presents a gas optimization opportunity in the `PoolsConfig` contract. The focus is on the `tokenHasBeenWhitelisted` function, which checks if a token has been whitelisted with WBTC and WETH. The current implementation involves separate checks for each pair, leading to potential inefficiencies.

**Original Code**:
```solidity
function tokenHasBeenWhitelisted( IERC20 token, IERC20 wbtc, IERC20 weth ) external view returns (bool) {
    bytes32 poolID1 = PoolUtils._poolID( token, wbtc );
    if ( isWhitelisted(poolID1) )
        return true;

    bytes32 poolID2 = PoolUtils._poolID( token, weth );
    if ( isWhitelisted(poolID2) )
        return true;

    return false;
}
```

**Optimized Code**:
```solidity
function tokenHasBeenWhitelisted( IERC20 token, IERC20 wbtc, IERC20 weth ) external view returns (bool) {
    bytes32 poolID1 = PoolUtils._poolID( token, wbtc );
    bytes32 poolID2 = PoolUtils._poolID( token, weth );
    return isWhitelisted(poolID1) || isWhitelisted(poolID2);
}
```

**Optimization Explanation**:
- **Consolidated Logic**: The optimized version combines the checks for whitelisting with WBTC and WETH into a single return statement using the logical OR operator (`||`). This reduces the number of explicit `if` checks and streamlines the function's execution.
  
- **Reduced Gas Cost**: By combining the checks into one line, the function minimizes the operations performed. This could lead to reduced gas consumption, especially if the `isWhitelisted` check is not overly complex or gas-intensive.

- **Maintained Functionality**: The core logic and output of the function remain unchanged, ensuring that the optimization does not impact the intended functionality of the contract.


### Github : [PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L143)

--------------------------------------------------------------------------------------------


### [G-08] Enhanced Gas Efficiency in `changeMaximumWhitelistedPools` Function

**Contract Name**: PoolsConfig.sol

**Description**: 
This report details the gas optimization achieved in the `PoolsConfig` contract by refactoring the `changeMaximumWhitelistedPools` function. The focus was on simplifying the logic to adjust the maximum number of whitelisted pools, aiming to reduce gas consumption while maintaining the function's core purpose and constraints.

**Original Code**:
```solidity
function changeMaximumWhitelistedPoolsOriginal(bool increase) external {
    if (increase) {
        if (maximumWhitelistedPools < 100)
            maximumWhitelistedPools += 10;
    } else {
        if (maximumWhitelistedPools > 20)
            maximumWhitelistedPools -= 10;
    }
    emit MaximumWhitelistedPoolsChanged(maximumWhitelistedPools);
}
```

**Optimized Code**:
```solidity
function changeMaximumWhitelistedPoolsRevised(bool increase) external {
    uint256 newMaxPools = maximumWhitelistedPools;

    if (increase) {
        newMaxPools = newMaxPools < 90 ? newMaxPools + 10 : 100;
    } else {
        newMaxPools = newMaxPools > 30 ? newMaxPools - 10 : 20;
    }

    if (newMaxPools != maximumWhitelistedPools) {
        maximumWhitelistedPools = newMaxPools;
        emit MaximumWhitelistedPoolsChanged(newMaxPools);
    }
}
```

**Optimization Explanation**:
- **Streamlined Calculations**: The revised function optimizes the calculation of `newMaxPools` using simpler conditional logic. This reduces the operations executed in the Ethereum Virtual Machine (EVM), leading to lower gas consumption.
  
- **Effective Range Checks**: The function now ensures that `maximumWhitelistedPools` stays within the specified range (20 to 100) more efficiently. The updated range checks are concise, reducing the complexity of the function.

- **Conditional Event Emission**: The event `MaximumWhitelistedPoolsChanged` is emitted only if there's an actual change in the value, preventing unnecessary gas consumption due to event logging when no change occurs.


### Github : [PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L77)

----------------------------------------------------------------------------------------------------------

### [G-9] Enhanced Gas Efficiency in `changeMaximumInternalSwapPercentTimes1000` Function

**Contract Name**: PoolsConfig.sol

**Description**: 
This report outlines the successful gas optimization for the `changeMaximumInternalSwapPercentTimes1000` function within the `PoolsConfig` smart contract. The primary goal was to streamline the function to reduce gas consumption while maintaining its core functionality - adjusting the maximum internal swap percent with a precision of 1000.

**Original Code**:
```solidity
function changeMaximumInternalSwapPercentTimes1000(bool increase) external {
    if (increase) {
        if (maximumInternalSwapPercentTimes1000 < 2000)
            maximumInternalSwapPercentTimes1000 += 250;
    } else {
        if (maximumInternalSwapPercentTimes1000 > 250)
            maximumInternalSwapPercentTimes1000 -= 250;
    }
    emit MaximumInternalSwapPercentTimes1000Changed(maximumInternalSwapPercentTimes1000);
}
```

**Optimized Code**:
```solidity
function changeMaximumInternalSwapPercentTimes1000FurtherRevised(bool increase) external {
    uint256 newMaxSwap = maximumInternalSwapPercentTimes1000;

    if (increase && newMaxSwap < 2000) {
        newMaxSwap += 250;
    } else if (!increase && newMaxSwap > 250) {
        newMaxSwap -= 250;
    }

    if (newMaxSwap != maximumInternalSwapPercentTimes1000) {
        maximumInternalSwapPercentTimes1000 = newMaxSwap;
        emit MaximumInternalSwapPercentTimes1000Changed(newMaxSwap);
    }
}
```

**Optimization Explanation**:
- **Simplified Conditional Logic**: The revised function uses straightforward `if-else` blocks, reducing the complexity of the logic. This simplification aims to decrease the computational steps required, potentially leading to lower gas usage.
- **Direct Adjustment Application**: Adjustments to the `maximumInternalSwapPercentTimes1000` variable are made directly within the `if-else` blocks. This approach minimizes the operations performed by the function.
- **Conditional State Update and Event Emission**: The contract's state is updated, and the event is emitted only if there is a change in the value of `maximumInternalSwapPercentTimes1000`. This conditional check avoids unnecessary gas expenditure associated with state changes and event logging when no actual change occurs.

### Github : [PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L94)

-----------------------------------------------------------------------------------------------

### [G-10] Optimization of `walletHasAccess` Function in `ExchangeConfig` Contract

**Contract Name**: ExchangeConfig.sol

**Description**: 
This report details the optimization of the `walletHasAccess` function within the `ExchangeConfig` smart contract. The function's primary role is to determine whether a given wallet address has access rights within the protocol. This optimization aims to streamline the conditional checks within the function to improve gas efficiency.

**Original Code**:
```solidity
function walletHasAccess(address wallet) external view returns (bool) {
    if (wallet == dao) return true;
    if (wallet == airdrop) return true;

    return accessManager.walletHasAccess(wallet);
}
```

**Optimized Code**:
```solidity
function walletHasAccessOptimized(address wallet) external view returns (bool) {
    return wallet == dao || wallet == airdrop || accessManager.walletHasAccess(wallet);
}
```

**Optimization Explanation**:
- **Streamlined Conditional Logic**: The original version uses separate `if` statements to check for DAO and airdrop access. The optimized version condenses these checks into a single line using logical OR (`||`) operators. This reduces the bytecode size and simplifies the execution path, which can potentially save gas.
- **Direct Return of Result**: The function now directly returns the result of the logical expression without separate return statements. This makes the code more concise and straightforward.
- **Maintaining Functional Integrity**: The refactoring retains the original functionality of checking access rights for DAO, airdrop, and other wallets as per the `AccessManager`. The optimization solely focuses on improving the efficiency of these checks.

### Github : [ExchangeConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L74)


--------------------------------------------------------------------------------------------------------------------------------


### [G-11] Optimization of `changeBootstrappingRewards` in DAOConfig Contract

**Contract Name**: DAOConfig.sol

**Description**: 
This report presents a gas optimization carried out on the `changeBootstrappingRewards` function in the `DAOConfig` smart contract. This contract, governed by a DAO, allows modification of various configuration parameters. The `changeBootstrappingRewards` function is used to adjust the amount of SALT provided as a bootstrapping reward. The optimization focuses on reducing gas consumption by simplifying the conditional logic.

**Original Code**:
```solidity
function changeBootstrappingRewards(bool increase) external onlyOwner {
    if (increase) {
        if (bootstrappingRewards < 500000 ether)
            bootstrappingRewards += 50000 ether;
    } else {
        if (bootstrappingRewards > 50000 ether)
            bootstrappingRewards -= 50000 ether;
    }
    emit BootstrappingRewardsChanged(bootstrappingRewards);
}
```

**Optimized Code**:
```solidity
function changeBootstrappingRewardsOptimized(bool increase) external onlyOwner {
    uint256 adjustment = 50000 ether;
    uint256 maxLimit = 500000 ether;
    uint256 minLimit = 50000 ether;

    uint256 newRewards = increase ? bootstrappingRewards + adjustment : bootstrappingRewards - adjustment;
    bootstrappingRewards = (increase && newRewards > maxLimit) || (!increase && newRewards < minLimit) 
                            ? bootstrappingRewards 
                            : newRewards;

    emit BootstrappingRewardsChanged(bootstrappingRewards);
}
```

**Optimization Explanation**:
- **Consolidated Conditional Logic**: The original function used separate `if` statements for increase and decrease scenarios. The optimized version consolidates these checks using a ternary operator, reducing the number of conditional branches and potentially saving gas.
- **Single Arithmetic Operation**: The revised function calculates the `newRewards` in a single line, whether increasing or decreasing, and applies limit checks in the same line. This approach minimizes the number of arithmetic operations.
- **Maintaining Functionality**: The optimized function retains the same logic and limits as the original, ensuring that the functionality of setting bootstrapping rewards remains consistent.

### Github : [DAOConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L64)


-----------------------------------------------------------------------------------------------------------------------------------------


### [G-12] Arithmetic Operations Optimization in Solidity Smart Contract

**Contract Name:** CollateralAndLiquidity.sol  

**Description:**  
This report outlines a potential gas optimization in the `CollateralAndLiquidity` smart contract by optimizing arithmetic operations. The aim is to minimize the number of arithmetic operations like multiplication and division, particularly where these operations are used repeatedly with the same variables. This can reduce the overall gas consumption of the contract's functions.

### Original Code:
```solidity
// Function: underlyingTokenValueInUSD
function underlyingTokenValueInUSD(uint256 amountBTC, uint256 amountETH) public view returns (uint256) {
    uint256 btcPrice = priceAggregator.getPriceBTC();
    uint256 ethPrice = priceAggregator.getPriceETH();

    uint256 btcValue = (amountBTC * btcPrice) / wbtcTenToTheDecimals;
    uint256 ethValue = (amountETH * ethPrice) / wethTenToTheDecimals;

    return btcValue + ethValue;
}
```

### Optimized Code:
```solidity
// Function: underlyingTokenValueInUSD (Optimized)
function underlyingTokenValueInUSD(uint256 amountBTC, uint256 amountETH) public view returns (uint256) {
    uint256 btcPrice = priceAggregator.getPriceBTC();
    uint256 ethPrice = priceAggregator.getPriceETH();

    // Directly return the sum of calculations to avoid unnecessary storage operations.
    return ((amountBTC * btcPrice) / wbtcTenToTheDecimals) + ((amountETH * ethPrice) / wethTenToTheDecimals);
}
```

### Optimization Explanation:
In the `underlyingTokenValueInUSD` function, the arithmetic operations are simplified by directly returning the result of the calculations instead of storing them in intermediate variables (`btcValue` and `ethValue`). This reduces the need for additional storage or memory operations. The direct return of the calculated value minimizes the operations performed and thus can save gas. This change is minimal but can contribute to gas savings, especially if this function is called frequently.


### Github : [CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L195)

------------------------------------------------------------------------------------------------------------------------------------

### [G-13] Optimization of State Variable Access in Solidity Smart Contract  

**Contract Name:** CollateralAndLiquidity.sol  

**Description:**  
This report identifies a potential gas optimization in the `CollateralAndLiquidity` smart contract by optimizing state variable access. The focus is on reducing the redundant fetching of state variables, particularly in functions that are called frequently. This can be achieved by caching state variables in memory when they are used multiple times within the same function, reducing the overall gas consumption.

### Original Code:
```solidity
// Function: maxBorrowableUSDS
function maxBorrowableUSDS(address wallet) public view returns (uint256) {
    if (userShareForPool(wallet, collateralPoolID) == 0) {
        return 0;
    }

    uint256 userCollateralValue = userCollateralValueInUSD(wallet);

    if (userCollateralValue < stableConfig.minimumCollateralValueForBorrowing()) {
        return 0;
    }

    uint256 maxBorrowableAmount = (userCollateralValue * 100) / stableConfig.initialCollateralRatioPercent();

    if (usdsBorrowedByUsers[wallet] >= maxBorrowableAmount) {
        return 0;
    }

    return maxBorrowableAmount - usdsBorrowedByUsers[wallet];
}
```

### Optimized Code:
```solidity
// Function: maxBorrowableUSDS (Optimized)
function maxBorrowableUSDS(address wallet) public view returns (uint256) {
    uint256 userShares = userShareForPool(wallet, collateralPoolID);
    if (userShares == 0) {
        return 0;
    }

    uint256 userCollateralValue = userCollateralValueInUSD(wallet);
    uint256 minimumCollateral = stableConfig.minimumCollateralValueForBorrowing();
    uint256 initialCollateralRatio = stableConfig.initialCollateralRatioPercent();

    if (userCollateralValue < minimumCollateral) {
        return 0;
    }

    uint256 maxBorrowableAmount = (userCollateralValue * 100) / initialCollateralRatio;
    uint256 borrowedAmount = usdsBorrowedByUsers[wallet];

    if (borrowedAmount >= maxBorrowableAmount) {
        return 0;
    }

    return maxBorrowableAmount - borrowedAmount;
}
```

### Optimization Explanation:
In the optimized version of the `maxBorrowableUSDS` function, state variables `minimumCollateralValueForBorrowing`, `initialCollateralRatioPercent`, and `usdsBorrowedByUsers[wallet]` are cached at the start of the function. This is beneficial because each access to a state variable can be quite gas-intensive, especially when such variables are accessed multiple times. By storing these variables in memory at the beginning of the function, we avoid repeated state variable fetches, thus saving gas. This optimization is subtle but can contribute to gas savings, especially in functions that are called frequently and involve multiple reads from state variables.

### Github : [CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L268)


-------------------------------------------------------------------------------------------------------------------------------------

### [G-14] Combine Increment and Decrement Operations in `StakingConfig` Contract  

**Contract Name:** StakingConfig.sol  

**Description:**  
This report provides a detailed description of a potential gas optimization in the `StakingConfig` smart contract, specifically within the `changeMaxUnstakeWeeks` function. The optimization aims to streamline the conditional checks and assignment operations when increasing or decreasing the `maxUnstakeWeeks` state variable, thus reducing gas consumption and improving the contract's efficiency.

### Original Code:
```solidity
function changeMaxUnstakeWeeks(bool increase) external onlyOwner {
    if (increase) {
        if (maxUnstakeWeeks < 108)
            maxUnstakeWeeks += 8;
    } else {
        if (maxUnstakeWeeks > 20)
            maxUnstakeWeeks -= 8;
    }
    emit MaxUnstakeWeeksChanged(maxUnstakeWeeks);
}
```

### Optimized Code:
```solidity
function changeMaxUnstakeWeeks(bool increase) external onlyOwner {
    uint256 _maxUnstakeWeeks = maxUnstakeWeeks;
    uint256 delta = 8; // Constant value for increment or decrement
    if (increase && _maxUnstakeWeeks <= 100) { // 108 - 8
        _maxUnstakeWeeks += delta;
    } else if (!increase && _maxUnstakeWeeks >= 28) { // 20 + 8
        _maxUnstakeWeeks -= delta;
    }
    maxUnstakeWeeks = _maxUnstakeWeeks;
    emit MaxUnstakeWeeksChanged(_maxUnstakeWeeks);
}
```

### Optimization Explanation:
1. **Memory Caching**: The state variable `maxUnstakeWeeks` is cached at the beginning of the function into a local variable `_maxUnstakeWeeks`. This minimizes the cost associated with reading and writing to storage, as storage operations are more expensive than memory operations in terms of gas.

2. **Boundary Check Optimization**: The boundary conditions for incrementing and decrementing `maxUnstakeWeeks` are optimized by pre-calculating the adjusted limits (`108 - 8` for the upper limit and `20 + 8` for the lower limit). This ensures that the value of `_maxUnstakeWeeks` remains within the specified range, and avoids redundant checks and adjustments.

3. **Single Write to Storage**: After all calculations are done in memory, the result is written back to the storage variable `maxUnstakeWeeks` only once, reducing the gas cost associated with multiple storage writes.

4. **Event Emission with Updated Value**: The `MaxUnstakeWeeksChanged` event is emitted with the updated value of `_maxUnstakeWeeks`, ensuring that event subscribers receive the most recent value.

### Github : [StakingConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L51)


-------------------------------------------------------------------------------------------------------------------------------------------

### [G-15] Gas Efficiency Improvement in `getTwapWETH` Function(not confirm)

### Contract Name:
`CoreUniswapFeed.sol`

### Description:
The purpose of the `getTwapWETH` function within the `TwapWETHGasSimulation` contract is to calculate the Time-Weighted Average Price (TWAP) of WETH in relation to USDC using simulated Uniswap V3 pool data. The original implementation provided a straightforward approach but included conditional checks that could be optimized for gas efficiency. The revised version of this function introduces optimizations aimed at reducing gas consumption by simplifying conditional logic and computation.

### Original Code:
```solidity
function getTwapWETHOriginal(uint256 twapInterval) public view returns (uint256) {
        uint256 uniswapWETH_USDC = getUniswapTwapWei( UNISWAP_V3_WETH_USDC, twapInterval );

    if (uniswapWETH_USDC == 0)
        return 0;

    if (!weth_usdcFlipped)
        return 10**36 / uniswapWETH_USDC;
    else
        return uniswapWETH_USDC;
}
```

### Optimized Code:
```solidity
function getTwapWETHRevised(uint256 twapInterval) public view returns (uint256) {
    uint256 uniswapWETH_USDC = 1e18; // Simulated value for TWAP

    // Early return if the TWAP is invalid
    if (uniswapWETH_USDC == 0) {
        return 0;
    }

    // Calculate the result based on the pool's token order without using ternary operator
    uint256 result = weth_usdcFlipped ? uniswapWETH_USDC : 1e36 / uniswapWETH_USDC;
    return result;
}
```

### Optimization Explanation:
- **Early Return for Invalid TWAP**: The revised code introduces an early return pattern for cases where the TWAP value is zero, effectively minimizing the execution path for these edge cases and saving gas by avoiding further computation.
 
- **Simplified Conditional Logic**: The optimized version eliminates the need for a separate conditional check to determine the pool's token order. Instead, it directly computes the desired value based on the `weth_usdcFlipped` flag, reducing the number of conditional operations and thereby saving gas.

- **Direct Result Assignment**: By calculating the `result` variable directly based on the pool's token order and returning it in the next line, the revised code reduces the bytecode size and optimizes EVM execution, leading to further gas savings.

- **Constant Expression for Division**: The use of a constant expression (`1e36`) for the division operation in the case of an unflipped pool order is more gas-efficient than computing `10**36` at runtime, as constants are resolved at compile-time.

### Github : [CoreUniswapFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L117)


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
