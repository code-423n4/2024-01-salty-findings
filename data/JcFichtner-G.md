[GAS-01] Consider using a more dynamic approach to handle these configurations if the number of parameters grows significantly.

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol

In Parameters.sol contract, it has a enum ParameterTypes which defines various configuration parameters. This enum is used as a function parameter in _executeParameterChange to determine which configuration parameter should be adjusted. The function then uses a series of if-else statements to check the value of this enum and perform the corresponding action.

Here’s how this is currently implemented:

function _executeParameterChange(ParameterTypes parameterType, bool increase, ...) internal {
    if (parameterType == ParameterTypes.maximumWhitelistedPools) {
        // ...
    }
    else if (parameterType == ParameterTypes.maximumInternalSwapPercentTimes1000) {
        // ...
    }
    // more else-if conditions
}

While enums are a gas-efficient way to represent a set of constants (since they are internally represented as integers), the way you're using them in a large conditional structure can be less efficient. When the _executeParameterChange function is called, the Ethereum Virtual Machine (EVM) has to check each condition until it finds the matching one. This means the gas cost increases with the position of the matched case in the if-else chain. For rarely matched cases that are checked later, this can be quite inefficient.

Gas Optimization Alternative:

An alternative approach is to use a more dynamic pattern, like a mapping of function pointers, to handle these configurations. Solidity supports function types as first-class citizens, and you can store these in a mapping. This way, you can directly invoke the appropriate function without going through a series of condition checks.

Here’s a simplified example of how this might look:

contract Parameters {
    mapping(ParameterTypes => function(bool) internal) public parameterChangeFunctions;

    constructor() {
        parameterChangeFunctions[ParameterTypes.maximumWhitelistedPools] = changeMaximumWhitelistedPools;
        parameterChangeFunctions[ParameterTypes.maximumInternalSwapPercentTimes1000] = changeMaximumInternalSwapPercentTimes1000;
        // Initialize other functions
    }

    function changeMaximumWhitelistedPools(bool increase) internal {
        // Implementation
    }

    function changeMaximumInternalSwapPercentTimes1000(bool increase) internal {
        // Implementation
    }

    function _executeParameterChange(ParameterTypes parameterType, bool increase) internal {
        function(bool) internal changeFunction = parameterChangeFunctions[parameterType];
        if (changeFunction != null) {
            changeFunction(increase);
        }
    }
}

In this revised approach:

1 - We store function pointers in a mapping, where each enum value maps to a specific function.

2 - The _executeParameterChange function looks up the appropriate function in the mapping and calls it if it exists.

This approach can significantly reduce the gas cost, especially when the contract has many configuration parameters. It eliminates the need for the EVM to sequentially check each condition, replacing it with a direct function call.
However, this pattern also has its trade-offs. It increases the complexity of the contract.

[GAS-02]  Consider batch processing multiple parameters in a single transaction to optimize gas usage.

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol

The _executeParameterChange function makes several external calls to different contract interfaces based on the ParameterTypes enum value. Each if-else branch within this function makes a call to an external contract method such as poolsConfig.changeMaximumWhitelistedPools(increase).

Current Implementation and Its Implications:

function _executeParameterChange(ParameterTypes parameterType, bool increase, ...) internal {
    if (parameterType == ParameterTypes.maximumWhitelistedPools) {
        poolsConfig.changeMaximumWhitelistedPools(increase);
    }
    // ... other else-if branches
}

Each of these calls is an external call, which can be quite gas-intensive in Ethereum. External calls are expensive because they involve a context switch as the EVM shifts control from one contract to another. Additionally, these calls require the transmission of information across contract boundaries and potentially involve state changes in multiple contracts.

Gas Optimization Strategy:

In scenarios where multiple parameters are often changed together, you could optimize these operations by batch processing the changes in a single transaction. This approach reduces the number of external calls, thus saving gas.

Here’s how you might implement this:

1 - Batch Update Function: Introduce a new function in your contract that can handle multiple parameter changes in one call.

2 - Data Structuring: Instead of passing a single parameter type and a boolean, you would pass arrays of parameter types and their corresponding desired states.

function batchExecuteParameterChanges(ParameterTypes[] memory parameterTypes, bool[] memory increases) public {
    require(parameterTypes.length == increases.length, "Arrays must be of the same length");

    for (uint i = 0; i < parameterTypes.length; i++) {
        _executeParameterChange(parameterTypes[i], increases[i]);
    }
}

In this approach:

1 - The batchExecuteParameterChanges function takes arrays of parameter types and their corresponding increase/decrease boolean values.

2 - It iterates over these arrays, calling _executeParameterChange for each element.

This batch processing means that if you need to change multiple parameters, you can do it in one transaction instead of multiple transactions, significantly reducing the total gas cost.

Considerations:

1 - Security: Ensure that this batch function is adequately protected, especially if it’s made public or external, to prevent unauthorized access.

2 - Complexity: This approach increases the complexity of the contract. Always balance gas optimization with contract simplicity and maintainability.
By applying this strategy, you optimize the contract for scenarios where multiple parameters are frequently adjusted together, thus saving gas over multiple transactions.

[GAS-03] Abstracting the increase/decrease logic into a generic internal function.

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol

In functions like changeBootstrappingRewards and other update functions, the pattern if (increase) {...} else {...} is repeated. If the logic for increasing/decreasing is similar across functions, consider abstracting this logic into an internal function to reduce code duplication and potentially save gas.

Let's create an internal function named _adjustParameter that will handle the logic for incrementing or decrementing. This function will take the current value, the allowed maximum and minimum values, and the adjustment amount as parameters. It will return the new adjusted value. We'll then modify the existing functions to use this new internal function.

Here's how the refactored code might look:

// SPDX-License-Identifier: BUSL 1.1
pragma solidity =0.8.22;

// ... [rest of the code and imports] ...

contract DAOConfig is IDAOConfig, Ownable {
    // ... [variable definitions and event declarations] ...

    // Generic internal function to adjust parameters
    function _adjustParameter(uint256 current, uint256 step, uint256 min, uint256 max, bool increase) internal pure returns (uint256) {
        if (increase) {
            if (current < max) {
                return current + step;
            }
        } else {
            if (current > min) {
                return current - step;
            }
        }
        return current;
    }

    // Modified functions to use the new internal function
    function changeBootstrappingRewards(bool increase) external onlyOwner {
        bootstrappingRewards = _adjustParameter(bootstrappingRewards, 50000 ether, 50000 ether, 500000 ether, increase);
        emit BootstrappingRewardsChanged(bootstrappingRewards);
    }

    function changePercentPolRewardsBurned(bool increase) external onlyOwner {
        percentPolRewardsBurned = uint256(_adjustParameter(percentPolRewardsBurned, 5, 25, 75, increase));
        emit PercentPolRewardsBurnedChanged(percentPolRewardsBurned);
    }

    // ... [rest of the adjustment functions, similarly modified] ...
}

In this refactoring, each adjustment function now calls _adjustParameter with the appropriate parameters. This reduces code duplication, making it cleaner and potentially saving gas, as the increment/decrement logic is centralized in a single function.

[GAS-04] Optimizing _arbitragePath Function in the ArbitrageSearch.sol

https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol

Caching Addresses: The _arbitragePath function repeatedly call address(swapTokenIn) and address(swapTokenOut). Consider caching these values in local variables at the start of the function to save on gas costs of repeated external calls​​.

Here's a revised version of your function incorporating this optimization:

function _arbitragePath(IERC20 swapTokenIn, IERC20 swapTokenOut) internal view returns (IERC20 arbToken2, IERC20 arbToken3) {
    address swapTokenInAddress = address(swapTokenIn);
    address swapTokenOutAddress = address(swapTokenOut);

    // swap: WBTC->WETH
    // arb: WETH->WBTC->SALT->WETH
    if (swapTokenInAddress == address(wbtc) && swapTokenOutAddress == address(weth))
        return (wbtc, salt);

    // swap: WETH->WBTC
    // arb: WETH->SALT->WBTC->WETH
    if (swapTokenInAddress == address(weth) && swapTokenOutAddress == address(wbtc))
        return (salt, wbtc);

    // swap: WETH->swapTokenOut
    // arb: WETH->WBTC->swapTokenOut->WETH
    if (swapTokenInAddress == address(weth))
        return (wbtc, swapTokenOut);

    // swap: swapTokenIn->WETH
    // arb: WETH->swapTokenIn->WBTC->WETH
    if (swapTokenOutAddress == address(weth))
        return (swapTokenIn, wbtc);

    // swap: swapTokenIn->swapTokenOut
    // arb: WETH->swapTokenOut->swapTokenIn->WETH
    return (swapTokenOut, swapTokenIn);
}

By caching swapTokenInAddress and swapTokenOutAddress, the cost associated with accessing storage is incurred only once for each variable instead of repeatedly throughout the function. 

[GAS-05] Cache the ballot variable from proposals.ballotForID(ballotID) to avoid repeated storage reads.

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol

This pattern is observed in multiple functions (_executeSetContract, _finalizeApprovalBallot, etc.) where caching could be applied.

For the _finalizeParameterBallot function, optimizing storage operations and applying branchless logic can lead to significant gas savings.

Storage Reads and Writes Optimization
Caching Storage Variables:

1 - In the _finalizeParameterBallot function, you're calling proposals.ballotForID(ballotID) to fetch the ballot. This is potentially a storage read operation that can be costly if repeated.

2 - Cache the result of proposals.ballotForID(ballotID) in a local variable at the start of the function. Then, use this cached variable throughout the function instead of calling proposals.ballotForID(ballotID) multiple times.

3 - Apply this caching principle to other functions like _executeSetContract and _finalizeApprovalBallot where similar patterns of repeated storage reads are observed​​.

Original _finalizeParameterBallot Function:

function _finalizeParameterBallot(uint256 ballotID) internal {
    Ballot memory ballot = proposals.ballotForID(ballotID);
    Vote winningVote = proposals.winningParameterVote(ballotID);

    if (winningVote == Vote.INCREASE) {
        _executeParameterChange(ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator);
    } else if (winningVote == Vote.DECREASE) {
        _executeParameterChange(ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator);
    }

    proposals.markBallotAsFinalized(ballotID);
    emit BallotFinalized(ballotID, winningVote);
}

Optimized _finalizeParameterBallot Function:

function _finalizeParameterBallot(uint256 ballotID) internal {
    Ballot memory cachedBallot = proposals.ballotForID(ballotID);
    Vote winningVote = proposals.winningParameterVote(ballotID);

    // Branchless logic approach
    bool isIncrease = winningVote == Vote.INCREASE;
    bool isDecrease = winningVote == Vote.DECREASE;
    _executeParameterChange(ParameterTypes(cachedBallot.number1), isIncrease, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator);
    
    // Note: The function _executeParameterChange must be able to handle a scenario where both isIncrease and isDecrease are false.
    // This is a simplified example of branchless logic. Depending on the complexity of the conditionals and the functions they call, 
    // a more sophisticated approach may be required.

    proposals.markBallotAsFinalized(ballotID);
    emit BallotFinalized(ballotID, winningVote);
}

Explanation of Changes:

1 - Caching: The ballot variable is now cached in cachedBallot and used throughout the function. This reduces repeated storage reads.

2 - Branchless Logic:

. I've used boolean variables isIncrease and isDecrease to represent the conditions.

. The call to _executeParameterChange is made once, with the conditionals handled within the boolean expressions.

. Ensure _executeParameterChange can correctly handle cases where both isIncrease and isDecrease are false.

[GAS-06]  Optimizing the excludedCountries mapping in DAO.sol

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol

The excludedCountries mapping uses strings as keys. If the set of country codes is limited and known, consider using an enum or a smaller data type like bytes2 for country codes to optimize storage.

To optimize the excludedCountries mapping in your Solidity contract, you can replace the use of strings as keys with a more gas-efficient approach. Since country codes are typically standardized and have a fixed length, using a smaller data type such as bytes2 or an enum can be more efficient. Here's how you can apply this change:

Using bytes2 for Country Codes

1 - Change the Key Type in Mapping
Instead of using string for country codes, use bytes2. This type is sufficient to hold standard ISO 3166-1 alpha-2 country codes, which are two characters long.

mapping(bytes2 => bool) public excludedCountries;

2 - Modify Access and Assignment

When accessing or assigning values in the excludedCountries mapping, convert the country codes to bytes2. This can be done using explicit type conversion.

// Setting a country as excluded
excludedCountries[bytes2('US')] = true;

// Checking if a country is excluded
bool isExcluded = excludedCountries[bytes2(countryCode)];

Using an enum for Country Codes:

If the set of countries is limited and known, an enum can be even more efficient

1 - Define an enum with all the country codes:

enum CountryCode { US, CA, GB, CN, IN, PK, RU, AF, CU, IR, KP, SY, VE }

2 - Change the Mapping:

Use the enum as the key type in the mapping.

mapping(CountryCode => bool) public excludedCountries;

3 - Access and Assignment:

// Setting a country as excluded
excludedCountries[CountryCode.US] = true;

// Checking if a country is excluded
bool isExcluded = excludedCountries[CountryCode.US];

General Considerations

Gas Costs: Using bytes2 or an enum can reduce the gas cost associated with storage and access of the mapping, as they occupy less space than strings.
Maintainability: If the set of countries might change or expand in the future, consider the ease of maintaining these changes with enum versus bytes2.
By implementing these changes, the excludedCountries mapping in your contract should become more gas-efficient, contributing to overall optimization.

[GAS-07] MULTIPLICATION/DIVISION BY TWO SHOULD USE BIT SHIFTING

x * 2 is equal to x << 1 and x / 2 is equal to x >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas.

uint256 averagePrice = ( priceA + priceB ) / 2;

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L139

require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247

exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L265

[GAS-08] REDUNDANT ARRAY INITIALIZATION

Declaring Arrays in Solidity by default assigns all the elements to 0 values therefore any attempt to overwrite them again with 0 is redundant.

secondsAgo[1] = 0; // to (now)

https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54

[GAS-09] CODE OPTIMIZATION BY USING MAX AND MIN

The contract was using the expression 2**128 to calculate the maximum storage capacity of the data type. This both lacks readability and costs more gas.
2**128, 2**64, 2**32, 2**16,  2**8, 2**4, 2**2, 2**1: can be replaced by type(uint).max; or type(uint).min; which is more readable and will also save some gas.

Example: type(uint128).max

However, keep in mind that type(uint128).max; is equal to 2**128 - 1

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L119-L126

function _mostSignificantBit(uint256 x) internal pure returns (uint256 msb)
    	{
        if (x >= 2**128) { x >>= 128; msb += 128; }
        if (x >= 2**64) { x >>= 64; msb += 64; }
        if (x >= 2**32) { x >>= 32; msb += 32; }
        if (x >= 2**16) { x >>= 16; msb += 16; }
        if (x >= 2**8) { x >>= 8; msb += 8; }
        if (x >= 2**4) { x >>= 4; msb += 4; }
        if (x >= 2**2) { x >>= 2; msb += 2; }
        if (x >= 2**1) { x >>= 1; msb += 1; }
	    }

[GAS-10] VARIABLES DECLARED BUT NEVER USED

The contract PoolUtils has declared a variable STAKED_SALT but it is not used anywhere in the code. This represents dead code or missing logic.

Unused variables increase the contract's size and complexity, potentially leading to higher gas costs and a larger attack surface.

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L16

bytes32 constant public STAKED_SALT = bytes32(0);

[GAS-11] UNUSED IMPORTS

Having unused code or import statements incurs extra gas usage when deploying the contract.

import "../staking/interfaces/IStaking.sol";
import "../Upkeep.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L11
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L18

import "./rewards/interfaces/IRewardsEmitter.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L6

import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L6

import "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
import "openzeppelin-contracts/contracts/utils/math/Math.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L3
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L5

import "../interfaces/ISalt.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L6

import "openzeppelin-contracts/contracts/utils/math/Math.sol";
import "./PoolMath.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L6
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L13

import "./interfaces/IDAO.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L14

import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
import "./interfaces/IPools.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L3
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L5

import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L5

[GAS-12] SUPERFLUOUS EVENT FIELDS

block.timestamp and block.number are by default added to event information. Adding them manually costs extra gas.

emit POLWithdrawn(tokenA, tokenB, reclaimedA, reclaimedB);

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L378
