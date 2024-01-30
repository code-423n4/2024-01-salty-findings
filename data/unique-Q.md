## \[N-01\] Use underscores for number literals

```diff
-     uint256 public bootstrappingRewards = 200000 ether;
+      uint256 public bootstrappingRewards = 200_000 ether;
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L24

```diff
-	uint256 public maximumInternalSwapPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier
+	uint256 public maximumInternalSwapPercentTimes1000 = 1_000;  // Defaults to 1.0% with a 1000x multiplier
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L41

```diff
-	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000;
+	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3_000;
```

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price\\\_feed/PriceAggregator.sol#L29](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price%5C_feed/PriceAggregator.sol#L29)

```diff
-        uint256 public rewardsEmitterDailyPercentTimes1000 = 1000;  // Defaults to 1.0% with a 1000x multiplier
+        uint256 public rewardsEmitterDailyPercentTimes1000 = 1_000;  // Defaults to 1.0% with a 1000x multiplier
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsConfig.sol#L19

```diff
-       uint256 public constant MILLION_ETHER = 1000000 ether;
+        uint256 public constant MILLION_ETHER = 1_000_000 ether;
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Salt.sol#L12

## \[N-02\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-03\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol  
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## \[N-04\] Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor

receive function (if exists)

fallback function (if exists)

external

public

internal

private

within a grouping, place the view and pure functions last

## \[05\] Un-indexed Parameters in Events

Consider indexing parameters for events, serving as logs filter when looking for specifically wanted data. Up to three parameters in an event function can receive the attribute indexed which will cause the respective arguments to be treated as log topics instead of data. Here is one of the instances entailed:

&nbsp;

```
event LiquidityDeposited(address indexed user, address indexed tokenA, address indexed tokenB, uint256 amountA, uint256 amountB, uint256 addedLiquidity);
    event LiquidityWithdrawn(address indexed user, address indexed tokenA, address indexed tokenB, uint256 amountA, uint256 amountB, uint256 withdrawnLiquidity);
```

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L20C3-L22C1

&nbsp; https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L25C2-L38C110

## \[N-06\] Use named parameters for mapping type declarations

Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18.

`mapping(uint256 => bytes32) public liens;`

&nbsp;

```
File: src/dao/DAO.sol

67: mapping(string=>bool) public excludedCountries;
```

&nbsp;

```
File: src/launch/Airdrop.sol
29: mapping(address=>bool) public claimed;
```

\[N‑07\] Contracts should have full test coverage

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

&nbsp;

## \[N-08\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-9\] use **Chainlink VRF**

**https://docs.chain.link/vrf**

## \[S-01\] Generate perfect code headers every time

Description:  
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

&nbsp;\[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.