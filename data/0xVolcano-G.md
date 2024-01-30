# Gas report

## Table of contents

- [Gas report](#gas-report)
  - [Table of contents](#table-of-contents)
    - [In accordance to the sponsors requirements, this report only focuses on issues that save more than 100 Gas](#in-accordance-to-the-sponsors-requirements-this-report-only-focuses-on-issues-that-save-more-than-100-gas)
  - [Pack `lastUpkeepTimeEmissions` and `lastUpkeepTimeRewardsEmitters` together by reducing their size to `uint128` (Saves 1 SLOT: 2.1K Gas)](#pack-lastupkeeptimeemissions-and-lastupkeeptimerewardsemitters-together-by-reducing-their-size-to-uint128-saves-1-slot-21k-gas)
  - [Pack the following by reducing their size(Save 3 SLOTS: 6.3K Gas)](#pack-the-following-by-reducing-their-sizesave-3-slots-63k-gas)
  - [Pack `minUnstakeWeeks,maxUnstakeWeeks,minUnstakePercent` by reducing the sizes(Save 2 SLOTs: 4.2K Gas)](#pack-minunstakeweeksmaxunstakeweeksminunstakepercent-by-reducing-the-sizessave-2-slots-42k-gas)
  - [Pack `rewardPercentForCallingLiquidation` with `maxRewardValueForCallingLiquidation`  by reducing the size(Save 1 SLOT: 2.1K Gas)](#pack-rewardpercentforcallingliquidation-with-maxrewardvalueforcallingliquidation--by-reducing-the-sizesave-1-slot-21k-gas)
  - [Pack `minimumCollateralValueForBorrowing` with `initialCollateralRatioPercent`(Save 1 SLOT: 2.1K Gas)](#pack-minimumcollateralvalueforborrowing-with-initialcollateralratiopercentsave-1-slot-21k-gas)
  - [Declare immutables for `exchangeConfig.wbtc()` and `exchangeConfig.weth()`(Save 2351 Gas on average)](#declare-immutables-for-exchangeconfigwbtc-and-exchangeconfigwethsave-2351-gas-on-average)
  - [Reference the immutable variable `salt` instead of making the external call again(Save 546 Gas on average)](#reference-the-immutable-variable-salt-instead-of-making-the-external-call-againsave-546-gas-on-average)
  - [Use the already defined immutable variable(save 595 Gas on average)](#use-the-already-defined-immutable-variablesave-595-gas-on-average)
  - [Use the imutable variable defined in the constructor instead of calling the external function again( Save 110 Gas)](#use-the-imutable-variable-defined-in-the-constructor-instead-of-calling-the-external-function-again-save-110-gas)
  - [Define immutable variables for `wbtc` and `weth`(Save 253 Gas on average)](#define-immutable-variables-for-wbtc-and-wethsave-253-gas-on-average)
  - [Immutable variable already defined(Save 327 Gas on average)](#immutable-variable-already-definedsave-327-gas-on-average)
  - [Expensive operation inside a for loop(Save 640 Gas on average)](#expensive-operation-inside-a-for-loopsave-640-gas-on-average)
  - [Avoid making unnecessary external calls(Save 4285 Gas on average)](#avoid-making-unnecessary-external-callssave-4285-gas-on-average)
  - [Refactor the code to avoid unnecessary external calls(Save 1300 Gas on average)](#refactor-the-code-to-avoid-unnecessary-external-callssave-1300-gas-on-average)
  - [We should cache the result of a function instead of calling it twice(Save 178 Gas on average )](#we-should-cache-the-result-of-a-function-instead-of-calling-it-twicesave-178-gas-on-average-)
  - [Avoid reading state variables due to how the logic is executed(Save 1 SLOAD: 100 Gas)](#avoid-reading-state-variables-due-to-how-the-logic-is-executedsave-1-sload-100-gas)
  - [Prioritize validating cheap variables first](#prioritize-validating-cheap-variables-first)
    - [Cheaper to validate `block.timestamp >= completionTimestamp` first](#cheaper-to-validate-blocktimestamp--completiontimestamp-first)
    - [Validate `liquidityToRemove` before reading from state](#validate-liquiditytoremove-before-reading-from-state)
    - [Validate the function parameters before making the state reads](#validate-the-function-parameters-before-making-the-state-reads)
    - [Validate the function parameter first before making any state reads](#validate-the-function-parameter-first-before-making-any-state-reads)
    - [Validate `amountRepaid` before making any state read](#validate-amountrepaid-before-making-any-state-read)
    - [Refactor function to only make state assignments after parameter verification](#refactor-function-to-only-make-state-assignments-after-parameter-verification)


### In accordance to the sponsors requirements, this report only focuses on issues that save more than 100 Gas

**Note: The issues addressed here were not reported by the bot, for packing variables, notes explaining the how and why are included**

**Gas estimates are given by running included tests ``COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url http://x.x.x.x:yyy --gas-report`` but for packing issues, we can simply calculate from the number of SLOTs saved**


## Pack `lastUpkeepTimeEmissions` and `lastUpkeepTimeRewardsEmitters` together by reducing their size to `uint128` (Saves 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L63-L64
```solidity
File: /src/Upkeep.sol
63:        uint256 public lastUpkeepTimeEmissions;
64:        uint256 public lastUpkeepTimeRewardsEmitters;
```
Since this are timestamps, we can reduce the size to something like `uint128` and should still be big enough. We can see the evidence of them being timestamps in how they are assigned

```solidity
86:                lastUpkeepTimeEmissions = block.timestamp;
87:                lastUpkeepTimeRewardsEmitters = block.timestamp;
```

Reducing the size will save us one storage slot

```diff
diff --git a/src/Upkeep.sol b/src/Upkeep.sol
index 7587bde..407850a 100644
--- a/src/Upkeep.sol
+++ b/src/Upkeep.sol
@@ -60,8 +60,8 @@ contract Upkeep is IUpkeep, ReentrancyGuard
        IUSDS  immutable public usds;
        IERC20  immutable public dai;

-       uint256 public lastUpkeepTimeEmissions;
-       uint256 public lastUpkeepTimeRewardsEmitters;
+       uint128 public lastUpkeepTimeEmissions;
+       uint128 public lastUpkeepTimeRewardsEmitters;

```

## Pack the following by reducing their size(Save 3 SLOTS: 6.3K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L21-L34
```solidity
File: /src/price_feed/PriceAggregator.sol
21:        IPriceFeed public priceFeed3; // CoreSaltyFeed by default

23:        // The next time at which setPriceFeed can be called
24:        uint256 public priceFeedModificationCooldownExpiration;

28:        // Range: 1% to 7% with an adjustment of .50%
29:        uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

33:        // Range: 30 to 45 days with an adjustment of 5 days
34:        uint256 public priceFeedModificationCooldown = 35 days;
```

Since `priceFeedModificationCooldownExpiration` is a timestamp, we can reduce it's size to `uint48` which should be safe enough.
`maximumPriceFeedPercentDifferenceTimes1000` has a defined range with the max bound being `7000` therefore `uint32` can fit the max value
`priceFeedModificationCooldown` also has some ranges defined with the max value being 45 days we can therefore use `uint16` or even `uint8`

Note, all the bounds are enforced whenever we alter the variables , eg https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L65-L79
```solidity
        function changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase) public onlyOwner
                {
        if (increase)
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 < 7000)
                maximumPriceFeedPercentDifferenceTimes1000 += 500;
            }
        else
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 > 1000)
                maximumPriceFeedPercentDifferenceTimes1000 -= 500;
            }

```

Note, the value would only be incremented if we are currently less than 7000, which ensures that we never go above the 7000 bound.

The same enforcement is done for `priceFeedModificationCooldown` here https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L82-L96

We choose to pack with the address `priceFeed3` as they are all accessed within the same transaction

```diff
diff --git a/src/price_feed/PriceAggregator.sol b/src/price_feed/PriceAggregator.sol
index ab23f29..c987c0a 100644
--- a/src/price_feed/PriceAggregator.sol
+++ b/src/price_feed/PriceAggregator.sol
@@ -18,20 +18,20 @@ contract PriceAggregator is IPriceAggregator, Ownable

        IPriceFeed public priceFeed1; // CoreUniswapFeed by default
        IPriceFeed public priceFeed2; // CoreChainlinkFeed by default
-       IPriceFeed public priceFeed3; // CoreSaltyFeed by default
+       IPriceFeed public priceFeed3; // CoreSaltyFeed by default //@audit 20

        // The next time at which setPriceFeed can be called
-       uint256 public priceFeedModificationCooldownExpiration;
+       uint48 public priceFeedModificationCooldownExpiration;//@audit timestamp

        // The maximum percent difference between two non-zero PriceFeed prices when aggregating price.
        // When the two closest PriceFeeds (out of the three) have prices further apart than this the aggregated price is considered invalid.
        // Range: 1% to 7% with an adjustment of .50%
-       uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier
+       uint32 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier //@audit max = 7k

        // The required cooldown between calls to setPriceFeed.
        // Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
        // Range: 30 to 45 days with an adjustment of 5 days
-       uint256 public priceFeedModificationCooldown = 35 days;
+       uint16 public priceFeedModificationCooldown = 35 days; //@audit max 45
```



## Pack `minUnstakeWeeks,maxUnstakeWeeks,minUnstakePercent` by reducing the sizes(Save 2 SLOTs: 4.2K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingConfig.sol#L18-L26
```solidity
File: /src/staking/StakingConfig.sol
17:        // Range: 1 to 12 with an adjustment of 1
18:        uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

21:        // Range: 20 to 108 with an adjustment of 8
22:        uint256 public maxUnstakeWeeks = 52;

25:        // Range: 10 to 50 with an adjustment of 5
26:        uint256 public minUnstakePercent = 20;
```

We can reduce the size of the the variables above to `uint64` . This should be safe as the variables have some Ranges already defined.

For `minUnstakeWeeks` the max the value can get is 12 and this is also enforced on the function `changeMinUnstakeWeeks`
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingConfig.sol#L34-L48
```solidity
        function changeMinUnstakeWeeks(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minUnstakeWeeks < 12)
                minUnstakeWeeks += 1;
            }
        else
            {
            if (minUnstakeWeeks > 1)
                minUnstakeWeeks -= 1;
            }

                emit MinUnstakeWeeksChanged(minUnstakeWeeks);
        }
```
Note, the value is only incremented if it's less than 12

The other variables also have defined bounds , for `maxUnstakeWeeks` the biggest we can get is `108` while for `minUnstakePercent` the range is defined as `10` to `50`



```diff
diff --git a/src/staking/StakingConfig.sol b/src/staking/StakingConfig.sol
index 33b77d9..d94d08a 100644
--- a/src/staking/StakingConfig.sol
+++ b/src/staking/StakingConfig.sol
@@ -15,15 +15,15 @@ contract StakingConfig is IStakingConfig, Ownable

     // The minimum number of weeks for an unstake request at which point minUnstakePercent of the original staked SALT is reclaimable.
        // Range: 1 to 12 with an adjustment of 1
-       uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks
+       uint64 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

        // The maximum number of weeks for an unstake request at which point 100% of the original staked SALT is reclaimable.
        // Range: 20 to 108 with an adjustment of 8
-       uint256 public maxUnstakeWeeks = 52;
+       uint64 public maxUnstakeWeeks = 52;//@audit (Always bound between 20 -108)

        // The percentage of the original staked SALT that is reclaimable when unstaking the minimum number of weeks.
        // Range: 10 to 50 with an adjustment of 5
-       uint256 public minUnstakePercent = 20;
+       uint64 public minUnstakePercent = 20;
```

The reason we pack this variables together is because they are accessed in the same transaction shown below
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L198-L212
```solidity
        function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
                {
                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
                require(numWeeks <= maxUnstakeWeeks,"Unstaking duration too long");

                uint256 percentAboveMinimum = 100 - minUnstakePercent;
                uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;

                uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
        return numerator / ( 100 * unstakeRange );
                }
```

## Pack `rewardPercentForCallingLiquidation` with `maxRewardValueForCallingLiquidation`  by reducing the size(Save 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/StableConfig.sol#L19-L24
```solidity
File: /src/stable/StableConfig.sol
19:        // Range: 5 to 10 with an adjustment of 1
20:    uint256 public rewardPercentForCallingLiquidation = 5;

23:        // Range: 100 to 1000 with an adjustment of 100 ether
24:    uint256 public maxRewardValueForCallingLiquidation = 500 ether;
```

Since the above two are accessed in the same transaction here https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L157-L164
```solididity
                uint256 rewardPercent = stableConfig.rewardPercentForCallingLiquidation();


                uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
                uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;


                // Make sure the value of the rewardAmount is not excessive
                uint256 rewardValue = underlyingTokenValueInUSD( rewardedWBTC, rewardedWETH ); // in 18 decimals
                uint256 maxRewardValue = stableConfig.maxRewardValueForCallingLiquidation(); // 18 decimals
```

We can reduce the size from `uint256` to `uint128` and pack them together

`uint128` should be big enough since this variables are bound and have defined ranges that are enforced whenever we assign values to them. eg Whenever we intend to increment the value of `rewardPercentForCallingLiquidation` we make sure the current value is less than `10` which means we will never go above 10.

For `maxRewardValueForCallingLiquidation` the max value we can ever get is defined as `1000`

From the defined bounds `uint128` should be big enough

```diff
diff --git a/src/stable/StableConfig.sol b/src/stable/StableConfig.sol
index 8d6f5de..637833a 100644
--- a/src/stable/StableConfig.sol
+++ b/src/stable/StableConfig.sol
@@ -17,21 +17,21 @@ contract StableConfig is IStableConfig, Ownable

        // The reward (in collateraLP) that a user receives for instigating the liquidation process - as a percentage of the amount of collateralLP that is liquidated.
        // Range: 5 to 10 with an adjustment of 1
-    uint256 public rewardPercentForCallingLiquidation = 5;
+    uint128 public rewardPercentForCallingLiquidation = 5;

        // The maximum reward value (in USD) that a user receives for instigating the liquidation process.
        // Range: 100 to 1000 with an adjustment of 100 ether
-    uint256 public maxRewardValueForCallingLiquidation = 500 ether;
+    uint128 public maxRewardValueForCallingLiquidation = 500 ether;//@audit pack this and above
```


## Pack `minimumCollateralValueForBorrowing` with `initialCollateralRatioPercent`(Save 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/StableConfig.sol#L28-L34
```solidity
File: /src/stable/StableConfig.sol
28:        // Range: 1000 to 5000 with an adjustment of 500 ether
29:    uint256 public minimumCollateralValueForBorrowing = 2500 ether;

33:    // Range: 150 to 300 with an adjustment of 25
34:    uint256 public initialCollateralRatioPercent = 200;

38:        // Range: 110 to 120 with an adjustment of 1
39:        uint256 public minimumCollateralRatioPercent = 110;
```

The defined ranges can well fit in `uint128` and pack this together. The two are accessed in the same transaction thus we can benefit from packing them

```diff
        // Range: 1000 to 5000 with an adjustment of 500 ether
-    uint256 public minimumCollateralValueForBorrowing = 2500 ether;
+    uint128 public minimumCollateralValueForBorrowing = 2500 ether;

     // the initial maximum collateral / borrowed USDS ratio.
     // Defaults to 2.0x so that staking $1000 worth of BTC/ETH LP would allow you to borrow $500 of USDS
     // Range: 150 to 300 with an adjustment of 25
-    uint256 public initialCollateralRatioPercent = 200;
+    uint128 public initialCollateralRatioPercent = 200;
```


## Declare immutables for `exchangeConfig.wbtc()` and `exchangeConfig.weth()`(Save 2351 Gas on average)

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 10259 | 126914   | 13406  | 382471 
| After  | 7636 | 124563   | 12095  | 379848 
|        |      |         |        |       

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180-L191
```solidity
File: /src/dao/Proposals.sol
180:        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181:                {
182:                require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
183:                require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
184:                require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
```


```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..e66fadc 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -32,6 +32,8 @@ contract Proposals is IProposals, ReentrancyGuard
     IPoolsConfig immutable public poolsConfig;
     IDAOConfig immutable public daoConfig;
     ISalt immutable public salt;
+       IERC20 immutable public wbtc;
+       IERC20 immutable public weth;

        // Mapping from ballotName to a currently open ballotID (zero if none).
        // Used to check for existing ballots by name so as to not allow duplicate ballots to be created.
@@ -75,6 +77,8 @@ contract Proposals is IProposals, ReentrancyGuard
                daoConfig = _daoConfig;

                salt = exchangeConfig.salt();
+               wbtc = exchangeConfig.wbtc();
+               weth = exchangeConfig.weth();
         }


@@ -180,8 +184,8 @@ contract Proposals is IProposals, ReentrancyGuard
        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
                {
                require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
-               require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
-               require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
+               require( address(token) != address(wbtc), "Cannot unwhitelist WBTC" );
+               require( address(token) != address(weth), "Cannot unwhitelist WETH" );
                require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
                require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
                require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
```

## Reference the immutable variable `salt` instead of making the external call again(Save 546 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 5767 | 165956   | 167908  | 322904 |
| After  | 5767 | 165410   | 167253  | 322248 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L196-L209
```solidity
File: /src/dao/Proposals.sol
196:        function proposeSendSALT( address wallet, uint256 amount, string calldata description ) external nonReentrant returns (uint256 ballotID)
197:                {
198:                require( wallet != address(0), "Cannot send SALT to address(0)" );

201:                uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
202:                uint256 maxSendable = balance * 5 / 100;
203:                require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

207:                string memory ballotName = "sendSALT";
208:                return _possiblyCreateProposal( ballotName, BallotType.SEND_SALT, wallet, amount, "", description );
209:                }
```


we are making an external call `exchangeConfig.salt` but this call is already declared as an immutable variable and we should therefore reference the immutable variable `salt` instead


```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..b7e4b00 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -198,7 +200,7 @@ contract Proposals is IProposals, ReentrancyGuard
                require( wallet != address(0), "Cannot send SALT to address(0)" );

                // Limit to 5% of current balance
-               uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
+               uint256 balance = salt.balanceOf( address(dao) );
                uint256 maxSendable = balance * 5 / 100;
                require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

```

## Use the already defined immutable variable(save 595 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 1391 | 4323   | 3987  | 10399 |
| After  | 1391 | 3728   | 3332  | 9744 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L317-L334
```solidity
File: /src/dao/Proposals.sol
317:        function requiredQuorumForBallotType( BallotType ballotType ) public view returns (uint256 requiredQuorum)
318:                {

334:                uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
```
In the constructor, we have an immutable variable `salt` whose value is `exchangeConfig.salt()`

```diff
-               uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
+               uint256 totalSupply = ERC20(address(salt)).totalSupply();
                uint256 minimumQuorum = totalSupply * 5 / 1000;
```



## Use the imutable variable defined in the constructor instead of calling the external function again( Save 110 Gas)
**Gas benchmarks based on function `finalizeBallot()`

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 84993   | 50499  | 520797 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L166-L176
```solidity
File: /src/dao/DAO.sol
166:                else if ( ballot.ballotType == BallotType.SEND_SALT )
167:                        {

170:                        if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
171:                                {
172:                                IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );


174:                                emit SaltSent(ballot.address1, ballot.number1);
175:                                }
176:                        }
```

In this block, we are calling `exchangeConfig.salt()` two times. This is an external call and thus can be quite expensive. 
If we take a look at the constructor, we can see we already have this call defined in a variable `salt` 
`salt = exchangeConfig.salt();`
Instead of making the call again, we should just reference the variable `salt`


```diff
@@ -167,9 +167,9 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        {
                        // Make sure the contract has the SALT balance before trying to send it.
                        // This should not happen but is here just in case - to prevent approved proposals from reverting on finalization.
-                       if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
+                       if ( salt.balanceOf(address(this)) >= ballot.number1 )
                                {
-                               IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );
+                               IERC20(salt).safeTransfer( ballot.address1, ballot.number1 );

                                emit SaltSent(ballot.address1, ballot.number1);
                                }
```


**Similar instance**
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L246-L265
```solidity
File: /src/dao/DAO.sol
246:                        uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
247:                        require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );


265:                        exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
```


```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..c6d8878 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -243,7 +243,7 @@ contract DAO is IDAO, Parameters, ReentrancyGuard

                        // Make sure that the DAO contract holds the required amount of SALT for bootstrappingRewards.
                        // Twice the bootstrapping rewards are needed (for both the token/WBTC and token/WETH pools)
-                       uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
+                       uint256 saltBalance = salt.balanceOf( address(this) );
                        require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

                        // Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
@@ -262,7 +262,7 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
                        addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

-                       exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
+                       salt.approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
                        liquidityRewardsEmitter.addSALTRewards( addedRewards );

                        emit WhitelistToken(IERC20(ballot.address1));
```


## Define immutable variables for `wbtc` and `weth`(Save 253 Gas on average)

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7318 | 85131   | 50631  | 520885 |
| After  | 7252 | 84878   | 50455  | 517983 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L253-L258
```solidity
File: /src/dao/DAO.sol
253:                        // All tokens are paired with both WBTC and WETH, so whitelist both pairings
254:                        poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255:                        poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );


257:                        bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
258:                        bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
```


```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..daf47b1 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -57,6 +57,8 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
        ISalt immutable public salt;
        IUSDS immutable public usds;
        IERC20 immutable public dai;
+       IERC20 immutable public wbtc;
+       IERC20 immutable public weth;


        // The default IPFS URL for the website content (can be changed with a setWebsiteURL proposal)
@@ -85,6 +87,8 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
         usds = exchangeConfig.usds();
         salt = exchangeConfig.salt();
         dai = exchangeConfig.dai();
+        wbtc = exchangeConfig.wbtc();
+        weth = exchangeConfig.weth();

                // Gas saving approves for eventually forming Protocol Owned Liquidity
                salt.approve(address(collateralAndLiquidity), type(uint256).max);
@@ -251,11 +255,11 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

                        // All tokens are paired with both WBTC and WETH, so whitelist both pairings
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), wbtc );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), weth );

-                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
+                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), wbtc );
+                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), weth );

                        // Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
                        AddedReward[] memory addedRewards = new AddedReward[](2);
```


## Immutable variable already defined(Save 327 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 10259 | 126914   | 13406  | 382471 |
| After  | 10259 | 126587   | 13406  | 381816 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180-L191
```solidity
File: /src/dao/Proposals.sol
180:        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181:                {

187:                require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
```

In the constructor, we have an immutable variable `salt` whose value is `exchangeConfig.salt()`

```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..53964be 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -184,7 +184,7 @@ contract Proposals is IProposals, ReentrancyGuard
                require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
                require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
                require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
-               require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
+               require( address(token) != address(salt), "Cannot unwhitelist SALT" );
```


## Expensive operation inside a for loop(Save 640 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 677 | 15275   | 17655  | 22043 |
| After  | 677 | 14635   | 17019  | 20751 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L321-L327
```solidity
File: /src/stable/CollateralAndLiquidity.sol
321:                        for ( uint256 i = startIndex; i <= endIndex; i++ )
322:                                {
323:                                address wallet = _walletsWithBorrowedUSDS.at(i);

326:                                uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
```

Calling `stableConfig.minimumCollateralRatioPercent()` inside the loop is too expensive as this is an external call

```diff
diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
index fc14695..a7d0e6d 100644
--- a/src/stable/CollateralAndLiquidity.sol
+++ b/src/stable/CollateralAndLiquidity.sol
@@ -317,13 +317,14 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                uint256 totalCollateralShares = totalShares[collateralPoolID];
                uint256 totalCollateralValue = totalCollateralValueInUSD();

-               if ( totalCollateralValue != 0 )
+               if ( totalCollateralValue != 0 ){
+                       uint256 _minimumCollateralRatioPercent = stableConfig.minimumCollateralRatioPercent();
                        for ( uint256 i = startIndex; i <= endIndex; i++ )
                                {
                                address wallet = _walletsWithBorrowedUSDS.at(i);

                                // Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * _minimumCollateralRatioPercent) / 100;

                                // Determine minCollateral in terms of minCollateralValue
                                uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
```

## Avoid making unnecessary external calls(Save 4285 Gas on average)
**Gas benchmarks based on function finalizeBallot**
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 80718   | 43300  | 520797 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L113-L128
```solidity
File: /src/dao/DAO.sol
113:        function _finalizeParameterBallot( uint256 ballotID ) internal
114:                {
115:                Ballot memory ballot = proposals.ballotForID(ballotID);

117:                Vote winningVote = proposals.winningParameterVote(ballotID);

119:                if ( winningVote == Vote.INCREASE )
120:                        _executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
121:                else if ( winningVote == Vote.DECREASE )
122:                        _executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
```

The above function is called by `finalizeBallot` which has the following implementation
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L278-L292
```solidity
        function finalizeBallot( uint256 ballotID ) external nonReentrant
                {

				<--- Truncated-->
                Ballot memory ballot = proposals.ballotForID(ballotID);


                if ( ballot.ballotType == BallotType.PARAMETER )
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
                        _finalizeApprovalBallot(ballotID);
                }


```

Note, in both functions, we make a very similar external call `Ballot memory ballot = proposals.ballotForID(ballotID);`
Since, external calls can be quite expensive, inlining here would be the best solution so that we only make the external call once

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..7df6949 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -282,8 +282,19 @@ contract DAO is IDAO, Parameters, ReentrancyGuard

                Ballot memory ballot = proposals.ballotForID(ballotID);

-               if ( ballot.ballotType == BallotType.PARAMETER )
-                       _finalizeParameterBallot(ballotID);
+               if ( ballot.ballotType == BallotType.PARAMETER ){
+                       Vote winningVote = proposals.winningParameterVote(ballotID);
+
+                       if ( winningVote == Vote.INCREASE )
+                               _executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
+                       else if ( winningVote == Vote.DECREASE )
+                               _executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
+
+                       // Finalize the ballot even if NO_CHANGE won
+                       proposals.markBallotAsFinalized(ballotID);
+
+                       emit BallotFinalized(ballotID, winningVote);
+               }
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
```

## Refactor the code to avoid unnecessary external calls(Save 1300 Gas on average)
**Gas benchmarks based on function finalizeBallot**
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 83700   | 50444  | 520796 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L219-L228
```solidity
File: /src/dao/DAO.sol
219:        function _finalizeApprovalBallot( uint256 ballotID ) internal
220:                {
221:                if ( proposals.ballotIsApproved(ballotID ) )
222:                        {
223:                        Ballot memory ballot = proposals.ballotForID(ballotID);
224:                        _executeApproval( ballot );
225:                        }

227:                proposals.markBallotAsFinalized(ballotID);
228:                }
```

The function `_finalizeApprovalBallot` is called by `finalizeBallot` which has the following implementation

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L278-L291
```solidity
        function finalizeBallot( uint256 ballotID ) external nonReentrant
                {

                Ballot memory ballot = proposals.ballotForID(ballotID);


                if ( ballot.ballotType == BallotType.PARAMETER )
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
                        _finalizeApprovalBallot(ballotID);
                }
```

Both functions are making the same external call `Ballot memory ballot = proposals.ballotForID(ballotID);` which wastes too much gas. Inlining the call can save us one external call

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..aa3efe9 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -286,8 +286,14 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
-               else
-                       _finalizeApprovalBallot(ballotID);
+               else {
+                       if ( proposals.ballotIsApproved(ballotID ) )
+                       {
+                               _executeApproval( ballot );
+                       }
+
+                       proposals.markBallotAsFinalized(ballotID);
+               }
                }
```


## We should cache the result of a function instead of calling it twice(Save 178 Gas on average )
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 1102 | 69116   | 69740  | 78240 |
| After  | 1102 | 68938   | 69560  | 78060 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L56-L70
```solidity
File: /src/launch/Airdrop.sol
56:    function allowClaiming() external
57:        {

60:        require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

63:        uint256 saltBalance = salt.balanceOf(address(this));
64:                saltAmountForEachUser = saltBalance / numberAuthorized();
```

The function  `allowClaiming()` calls `numberAuthorized()` 2 times.
The function `numberAuthorized()` is implemented as follows
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L97-L100
```solidity
    function numberAuthorized() public view returns (uint256)
        {
        return _authorizedUsers.length();
        }
```
Note, it makes a state read(SLOAD) to get the length `_authorizedUsers.length()`
Doing this two times is expensive, we should simply save the results of this call and cache them

```diff
@@ -57,11 +57,12 @@ contract Airdrop is IAirdrop, ReentrancyGuard
        {
        require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
        require( ! claimingAllowed, "Claiming is already allowed" );
-               require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+               uint256 _numberAuthorized = numberAuthorized();
+               require(_numberAuthorized > 0, "No addresses authorized to claim airdrop.");

        // All users receive an equal share of the airdrop.
        uint256 saltBalance = salt.balanceOf(address(this));
-               saltAmountForEachUser = saltBalance / numberAuthorized();
+               saltAmountForEachUser = saltBalance / _numberAuthorized;

                // Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
                salt.approve( address(staking), saltBalance );
```

If we revert on the first check, then we would waste ~6 gas but in the case we don't revert, we save an entire SLOAD(100 Gas)

## Avoid reading state variables due to how the logic is executed(Save 1 SLOAD: 100 Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L73-L81
```solidity
File: /src/ManagedWallet.sol
73:        function changeWallets() external
74:                {
75:                // proposedMainWallet calls the function - to make sure it is a valid address.
76:                require( msg.sender == proposedMainWallet, "Invalid sender" );
77:                require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

79:                // Set the wallets
80:                mainWallet = proposedMainWallet;
81:                confirmationWallet = proposedConfirmationWallet;
```

Due to the require statement on line 76, it's guaranteed that `proposedMainWallet` will be equal to `msg.sender` if we do get to the `set the wallets` part. If they are not equal then the require statement would revert.
Therefore, any occurence of `proposedMainWallet` which is a state variable can be replaced by `msg.sender` which is a global variable therefore cheaper to read

```diff
diff --git a/src/ManagedWallet.sol b/src/ManagedWallet.sol
index 7082030..99c3ed0 100644
--- a/src/ManagedWallet.sol
+++ b/src/ManagedWallet.sol
@@ -77,10 +77,10 @@ contract ManagedWallet is IManagedWallet
                require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

                // Set the wallets
-               mainWallet = proposedMainWallet;
+               mainWallet = msg.sender;
                confirmationWallet = proposedConfirmationWallet;

-               emit WalletChange(mainWallet, confirmationWallet);
+               emit WalletChange(msg.sender, confirmationWallet);

                // Reset
                activeTimelock = type(uint256).max;
```


## Prioritize validating cheap variables first

### Cheaper to validate `block.timestamp >= completionTimestamp` first
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L69-L72
```solidity
File: /src/launch/BootstrapBallot.sol
69:        function finalizeBallot() external nonReentrant
70:                {
71:                require( ! ballotFinalized, "Ballot has already been finalized" );
72:                require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
```
The first check involves making an SLOAD to read the value of `ballotFinalized`. If this check passes but the second reverts, we waste the gas in the first check. We should reorder this checks to have the cheaper one first. 

```diff
        function finalizeBallot() external nonReentrant
                {
-               require( ! ballotFinalized, "Ballot has already been finalized" );
                require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
+               require( ! ballotFinalized, "Ballot has already been finalized" );
```


### Validate `liquidityToRemove` before reading from state
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L170-L173
```solidity
File: /src/pools/Pools.sol
170:        function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
171:                {
172:                require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
173:                require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
```

```diff
        function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
                {
-               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
                require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
+               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

                (bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);
```


### Validate the function parameters before making the state reads
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L45-L48
```solidity
File: /src/pools/PoolsConfig.sol
45:        function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
46:                {
47:                require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
48:                require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
```


```diff
        function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
                {
-               require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
                require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
+               require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );

```

### Validate the function parameter first before making any state reads
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L40-L43
```solidity
File: /src/stable/USDS.sol
40:        function mintTo( address wallet, uint256 amount ) external
41:                {
42:                require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
43:                require( amount > 0, "Cannot mint zero USDS" );
```

```diff
        function mintTo( address wallet, uint256 amount ) external
                {
-               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
                require( amount > 0, "Cannot mint zero USDS" );
+               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
```

### Validate `amountRepaid` before making any state read
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L115-L119
```solidity
File: /src/stable/CollateralAndLiquidity.sol
115:     function repayUSDS( uint256 amountRepaid ) external nonReentrant
116:                {
117:                require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
118:                require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
119:                require( amountRepaid > 0, "Cannot repay zero amount" );
```

```diff
      function repayUSDS( uint256 amountRepaid ) external nonReentrant
                {
+               require( amountRepaid > 0, "Cannot repay zero amount" );
                require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
                require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
-               require( amountRepaid > 0, "Cannot repay zero amount" );
```

### Refactor function to only make state assignments after parameter verification
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L198-L205
```solidity
File: /src/staking/Staking.sol
198:        function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
199:                {
200:                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
201:        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
202:        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

204:                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
205:                require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
```
The require statements only depend on the first two variables `minUnstakeWeeks` and `maxUnstakeWeeks`. We can therefore move the assignment of `minUnstakePercent` to be after all checks are passed.

```diff
diff --git a/src/staking/Staking.sol b/src/staking/Staking.sol
index 22e5970..c00c419 100644
--- a/src/staking/Staking.sol
+++ b/src/staking/Staking.sol
@@ -199,10 +199,12 @@ contract Staking is IStaking, StakingRewards
                {
                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
         uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
-        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
                require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
+
+        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();
+

                uint256 percentAboveMinimum = 100 - minUnstakePercent;
                uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;
```


