## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Unsafe downcast from larger to smaller integer types | 14 |
| [L-02] | Use of `ecrecover` is susceptible to signature malleability | 1 |
| [L-03] | Unbounded loop may run out of gas | 18 |
| [L-04] | Prevent Division by Zero | 3 |
| [L-05] | Functions calling tokens with transfer hooks are missing reentrancy guards | 8 |
| [L-06] | Consider implementing two-step procedure for updating protocol addresses | 13 |
| [L-07] | `internal/private` Function calls within for loops | 7 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | `require()`/`revert()` statements should have descriptive reason strings | 2 |
| [N-02] | `uint/int` variables should have the bit size defined explicitly | 2 |
| [N-03] | `if`-statement can be converted to a ternary | 27 |
| [N-04] | Avoid Empty Blocks in Code | 14 |
| [N-05] | Consider using OpenZeppelin's SafeCast for any casting | 32 |
| [N-06] | Leverage Recent Solidity Features with `0.8.23` | 35 |
| [N-07] | Mixed usage of `int/uint` with `int256/uint256` | 2 |
| [N-08] | Inconsistent spacing in comments | 2 |
| [N-09] | Function/Constructor Argument Names Not in mixedCase | 29 |
| [N-10] | Insufficient Comments in Complex Functions | 3 |
| [N-11] | Variable Names Not in mixedCase | 3 |
| [N-12] | Consider making contracts `Upgradeable` | 32 |
| [N-13] | Named Imports of Parent Contracts Are Missing | 50 |
| [N-14] | Long functions should be refactored into multiple, smaller, functions | 3 |
| [N-15] | Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments | 2 |
| [N-16] | Use Unchecked for Divisions on Constant or Immutable Values | 2 |



## Low Findings Details


### [L-01] Unsafe downcast from larger to smaller integer types

Casting from larger types to smaller ones can potentially lead to overflows and thus unexpected behavior.
For instance, when a `uint256` value gets cast to `uint8`, it could end up as an entirely different, much smaller number due to the reduction in bits. 

OpenZeppelin's `SafeCast` library provides functions for safe type conversions, throwing an error whenever an overflow would occur.
It is generally recommended to use `SafeCast` or similar protective measures when performing type conversions to ensure the accuracy of your computations and the security of your contracts.

<details>
<summary><i>14 issue instances in 3 files:</i></summary>

```solidity
File: src/pools/Pools.sol

/// @audit cast from `uint256` to `uint128`
100: reserves.reserve0 += uint128(maxAmount0);
/// @audit cast from `uint256` to `uint128`
101: reserves.reserve1 += uint128(maxAmount1);
/// @audit cast from `uint256` to `uint128`
125: reserves.reserve0 += uint128(addedAmount0);
/// @audit cast from `uint256` to `uint128`
126: reserves.reserve1 += uint128(addedAmount1);
/// @audit cast from `uint256` to `uint128`
182: reserves.reserve0 -= uint128(reclaimedA);
/// @audit cast from `uint256` to `uint128`
183: reserves.reserve1 -= uint128(reclaimedB);
```
[100](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L100) | [101](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L101) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L126) | [182](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L182) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L183)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

/// @audit cast from `uint256` to `uint32`
53: secondsAgo[0] = uint32(twapInterval); // from (before)
/// @audit casted from `int56` to `int24`
59: int24 tick = int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)));
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L53) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59)

```solidity
File: src/staking/StakingRewards.sol

/// @audit cast from `uint256` to `uint128`
83: user.virtualRewards += uint128(virtualRewardsToAdd);
/// @audit cast from `uint256` to `uint128`
84: totalRewards[poolID] += uint128(virtualRewardsToAdd);
/// @audit cast from `uint256` to `uint128`
88: user.userShare += uint128(increaseShareAmount);
/// @audit cast from `uint256` to `uint128`
125: user.userShare -= uint128(decreaseShareAmount);
/// @audit cast from `uint256` to `uint128`
126: user.virtualRewards -= uint128(virtualRewardsToRemove);
/// @audit cast from `uint256` to `uint128`
159: userInfo[poolID].virtualRewards += uint128(pendingRewards);
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L84) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L88) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L126) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L159)
</details>


### [L-02] Use of `ecrecover` is susceptible to signature malleability

The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to replay attacks.
References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121.
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.
[Consider using OpenZeppelin’s ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) (which prevents this malleability) instead of the built-in function.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SigningTools.sol

26: ecrecover(messageHash, v, r, s)
```
[26](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L26)
</details>


### [L-03] Unbounded loop may run out of gas

Unbounded loops in smart contracts pose a risk because they iterate over an unknown number of elements, potentially consuming all available gas for a transaction.
This can result in unintended transaction failures. Gas consumption increases linearly with the number of iterations, and if it surpasses the gas limit, the transaction reverts, wasting the gas spent.
To mitigate this, developers should either set a maximum limit on loop iterations.

<details>
<summary><i>18 issue instances in 5 files:</i></summary>

```solidity
File: src/dao/Proposals.sol

423: for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
```
[423](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423)

```solidity
File: src/pools/PoolStats.sol

53: for( uint256 i = 0; i < poolIDs.length; i++ )
65: for( uint256 i = 0; i < poolIDs.length; i++ )
81: for( uint256 i = 0; i < poolIDs.length; i++ )
106: for( uint256 i = 0; i < poolIDs.length; i++ )
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L53) | [65](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L65) | [81](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81) | [106](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L106)

```solidity
File: src/rewards/RewardsEmitter.sol

60: for( uint256 i = 0; i < addedRewards.length; i++ )
116: for( uint256 i = 0; i < poolIDs.length; i++ )
146: for( uint256 i = 0; i < rewards.length; i++ )
```
[60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L60) | [116](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L116) | [146](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L146)

```solidity
File: src/rewards/SaltRewards.sol

64: for( uint256 i = 0; i < addedRewards.length; i++ )
87: for( uint256 i = 0; i < addedRewards.length; i++ )
129: for( uint256 i = 0; i < poolIDs.length; i++ )
```
[64](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L64) | [87](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L87) | [129](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L129)

```solidity
File: src/staking/StakingRewards.sol

152: for( uint256 i = 0; i < poolIDs.length; i++ )
185: for( uint256 i = 0; i < addedRewards.length; i++ )
216: for( uint256 i = 0; i < shares.length; i++ )
226: for( uint256 i = 0; i < rewards.length; i++ )
260: for( uint256 i = 0; i < rewards.length; i++ )
277: for( uint256 i = 0; i < shares.length; i++ )
296: for( uint256 i = 0; i < cooldowns.length; i++ )
```
[152](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L152) | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185) | [216](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L216) | [226](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L226) | [260](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L260) | [277](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L277) | [296](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L296)
</details>


### [L-04] Prevent Division by Zero

On several locations in the code, precautions are not being taken for not dividing by 0. This will revert the code.
These functions can be called with a 0 value in the input, this value is not checked for being bigger than 0,
that means in some scenarios this can potentially trigger a division by zero.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: src/staking/StakingRewards.sol

/// @audit totalShars - can be zero
114: uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];
/// @audit usr - can be zero
118: uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;
/// @audit totalShars - can be zero
243: uint256 rewardsShare = ( totalRewards[poolID] * user.userShare ) / totalShares[poolID];
```
[114](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L114) | [118](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L118) | [243](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L243)
</details>


### [L-05] Functions calling tokens with transfer hooks are missing reentrancy guards

Functions that call contracts or addresses with transfer hooks should use reentrancy guards for protection. 
Even if these functions adhere to the check-effects-interaction best practice, absence of reentrancy guards can expose the protocol users to read-only reentrancies.
Without the guards, the only protective measure is to block-list the entire protocol, which isn't an optimal solution.

<details>
<summary><i>8 issue instances in 2 files:</i></summary>

```solidity
File: src/dao/DAO.sol

375: tokenA.safeTransfer( address(liquidizer), reclaimedA );
376: tokenB.safeTransfer( address(liquidizer), reclaimedB );
```
[375](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L375) | [376](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L376)

```solidity
File: src/staking/Liquidity.sol

/// @audit function `_depositLiquidityAndIncreaseShare()` 
88: tokenA.safeTransferFrom(msg.sender, address(this), maxAmountA );
/// @audit function `_depositLiquidityAndIncreaseShare()` 
89: tokenB.safeTransferFrom(msg.sender, address(this), maxAmountB );
/// @audit function `_depositLiquidityAndIncreaseShare()` 
111: tokenA.safeTransfer( msg.sender, maxAmountA - addedAmountA );
/// @audit function `_depositLiquidityAndIncreaseShare()` 
114: tokenB.safeTransfer( msg.sender, maxAmountB - addedAmountB );
/// @audit function `_withdrawLiquidityAndClaim()` 
131: tokenA.safeTransfer( msg.sender, reclaimedA );
/// @audit function `_withdrawLiquidityAndClaim()` 
132: tokenB.safeTransfer( msg.sender, reclaimedB );
```
[88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L88) | [89](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L89) | [111](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L111) | [114](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L114) | [131](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L131) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L132)
</details>


### [L-06] Consider implementing two-step procedure for updating protocol addresses

Direct state address changes in a function can be risky, as they don't allow for a verification step before the change is made.
It's safer to implement a two-step process where the new address is first proposed, then later confirmed, allowing for more control and the chance to catch errors or malicious activity.

<details>
<summary><i>13 issue instances in 4 files:</i></summary>

```solidity
File: src/pools/Pools.sol

/// @audit `dao` is changed
62: dao = _dao;
```
[62](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L62)

```solidity
File: src/stable/Liquidizer.sol

/// @audit `collateralAndLiquidity` is changed
65: collateralAndLiquidity = _collateralAndLiquidity;
/// @audit `pools` is changed
66: pools = _pools;
/// @audit `dao` is changed
67: dao = _dao;
```
[65](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L65) | [66](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L66) | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L67)

```solidity
File: src/ManagedWallet.sol

/// @audit `proposedMainWallet` is changed
50: proposedMainWallet = _proposedMainWallet;
/// @audit `proposedConfirmationWallet` is changed
52: proposedConfirmationWallet = _proposedConfirmationWallet;
```
[50](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L50) | [52](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L52)

```solidity
File: src/ExchangeConfig.sol

/// @audit `dao` is changed
52: dao = _dao;
/// @audit `upkeep` is changed
54: upkeep = _upkeep;
/// @audit `initialDistribution` is changed
55: initialDistribution = _initialDistribution;
/// @audit `airdrop` is changed
56: airdrop = _airdrop;
/// @audit `teamVestingWallet` is changed
57: teamVestingWallet = _teamVestingWallet;
/// @audit `daoVestingWallet` is changed
58: daoVestingWallet = _daoVestingWallet;
/// @audit `accessManager` is changed
65: accessManager = _accessManager;
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L52) | [54](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L54) | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L55) | [56](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L56) | [57](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L57) | [58](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L58) | [65](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L65)
</details>


### [L-07] `internal/private` Function calls within for loops

Making function calls or external calls within loops in Solidity can lead to inefficient gas usage, potential bottlenecks, and increased vulnerability to attacks.
Each function call or external call consumes gas, and when executed within a loop, the gas cost multiplies, potentially causing the transaction to run out of gas or exceed block gas limits.
This can result in transaction failure or unpredictable behavior.

<details>
<summary><i>7 issue instances in 4 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

120: _rightMoreProfitable( midpoint, reservesA0, reservesA1, reservesB0, reservesB1, reservesC0, reservesC1 )
```
[120](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L120)

```solidity
File: src/pools/PoolStats.sol

89: _poolIndex( _weth, arbToken2, poolIDs )
90: _poolIndex( arbToken2, arbToken3, poolIDs )
91: _poolIndex( arbToken3, _weth, poolIDs )
```
[89](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L89) | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L90) | [91](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L91)

```solidity
File: src/stable/CollateralAndLiquidity.sol

332: userShareForPool( wallet, collateralPoolID )
```
[332](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L332)

```solidity
File: src/staking/StakingRewards.sol

156: userRewardForPool( msg.sender, poolID )
261: userRewardForPool( wallet, poolIDs[i] )
```
[156](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L156) | [261](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L261)
</details>


## NonCritical Findings Details


### [N-01] `require()`/`revert()` statements should have descriptive reason strings

In Solidity, require() and revert() functions can include an optional 'reason' string that describes what caused the function to fail. This string can be incredibly helpful during testing and debugging, as it provides more context for what went wrong.

Please ensure that the reason strings are comprehensive and descriptive. Strings with fewer than 12 characters may not provide enough context to understand the cause of the function failure.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/pools/Pools.sol

/// @audit Reason string `TX EXPIRED` is not informative enough
83: require(block.timestamp <= deadline, "TX EXPIRED");
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L83)

```solidity
File: src/staking/Liquidity.sol

/// @audit Reason string `TX EXPIRED` is not informative enough
42: require(block.timestamp <= deadline, "TX EXPIRED");
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L42)
</details>


### [N-02] `uint/int` variables should have the bit size defined explicitly

In Solidity, int256 and uint256 are the preferred type names, especially since they are used for function signatures.
Using int or uint instead can lead to confusion and inconsistency.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/pools/Pools.sol

81: modifier ensureNotExpired(uint deadline)
```
[81](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L81)

```solidity
File: src/staking/Liquidity.sol

40: modifier ensureNotExpired(uint deadline)
```
[40](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L40)
</details>


### [N-03] `if`-statement can be converted to a ternary

The ternary operator provides a shorthand way to write if / else statements.
It can improve readability and reduce the number of lines of code.
Traditional `if / else` statements, particularly those spanning only a one lines, could potentially be refactored using the ternary operator.

<details>
<summary><i>27 issue instances in 10 files:</i></summary>

```solidity
File: src/dao/DAOConfig.sol

66: if (increase)
        	{
            if (bootstrappingRewards < 500000 ether)
                bootstrappingRewards += 50000 ether;
            }
       	 else
       	 	{
            if (bootstrappingRewards > 50000 ether)
                bootstrappingRewards -= 50000 ether;
	        }
83: if (increase)
			{
			if (percentPolRewardsBurned < 75)
				percentPolRewardsBurned += 5;
			}
		else
			{
			if (percentPolRewardsBurned > 25)
				percentPolRewardsBurned -= 5;
			}
100: if (increase)
			{
			if (baseBallotQuorumPercentTimes1000 < 20 * 1000)
				baseBallotQuorumPercentTimes1000 += 1000;
			}
		else
			{
			if (baseBallotQuorumPercentTimes1000 > 5 * 1000 )
				baseBallotQuorumPercentTimes1000 -= 1000;
			}
117: if (increase)
        	{
            if (ballotMinimumDuration < 14 days)
                ballotMinimumDuration += 1 days;
        	}
        else
        	{
            if (ballotMinimumDuration > 3 days)
                ballotMinimumDuration -= 1 days;
        	}
151: if (increase)
			{
			if (maxPendingTokensForWhitelisting < 12)
				maxPendingTokensForWhitelisting += 1;
			}
		else
			{
			if (maxPendingTokensForWhitelisting > 3)
				maxPendingTokensForWhitelisting -= 1;
			}
168: if (increase)
            {
            if (arbitrageProfitsPercentPOL < 45)
                arbitrageProfitsPercentPOL += 5;
            }
        else
            {
            if (arbitrageProfitsPercentPOL > 15)
                arbitrageProfitsPercentPOL -= 5;
            }
185: if (increase)
            {
            if (upkeepRewardPercent < 10)
                upkeepRewardPercent += 1;
            }
        else
            {
            if (upkeepRewardPercent > 1)
                upkeepRewardPercent -= 1;
            }
```
[66](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L66) | [83](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L83) | [100](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L100) | [117](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L117) | [151](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L151) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L168) | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L185)

```solidity
File: src/dao/Proposals.sol

267: if ( ballot.ballotType == BallotType.PARAMETER )
			require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
		else // If a Ballot is not a Parameter Ballot, it is an Approval ballot
			require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
```
[267](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L267)

```solidity
File: src/pools/PoolsConfig.sol

79: if (increase)
            {
            if (maximumWhitelistedPools < 100)
                maximumWhitelistedPools += 10;
            }
        else
            {
            if (maximumWhitelistedPools > 20)
                maximumWhitelistedPools -= 10;
            }
96: if (increase)
            {
            if (maximumInternalSwapPercentTimes1000 < 2000)
                maximumInternalSwapPercentTimes1000 += 250;
            }
        else
            {
            if (maximumInternalSwapPercentTimes1000 > 250)
                maximumInternalSwapPercentTimes1000 -= 250;
            }
```
[79](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L79) | [96](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L96)

```solidity
File: src/pools/Pools.sol

131: if ( addedAmount0 > addedAmount1)
			addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
		else
			addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
```
[131](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L131)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

44: if ( answerDelay <= MAX_ANSWER_DELAY )
				price = _price;
			else
				price = 0;
```
[44](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L44)

```solidity
File: src/price_feed/PriceAggregator.sol

67: if (increase)
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 < 7000)
                maximumPriceFeedPercentDifferenceTimes1000 += 500;
            }
        else
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 > 1000)
                maximumPriceFeedPercentDifferenceTimes1000 -= 500;
            }
84: if (increase)
            {
            if (priceFeedModificationCooldown < 45 days)
                priceFeedModificationCooldown += 5 days;
            }
        else
            {
            if (priceFeedModificationCooldown > 30 days)
                priceFeedModificationCooldown -= 5 days;
            }
```
[67](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L67) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L84)

```solidity
File: src/rewards/RewardsConfig.sol

38: if (increase)
            {
            if (rewardsEmitterDailyPercentTimes1000 < 2500)
                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 + 250;
            }
        else
            {
            if (rewardsEmitterDailyPercentTimes1000 > 250)
                rewardsEmitterDailyPercentTimes1000 = rewardsEmitterDailyPercentTimes1000 - 250;
            }
54: if (increase)
            {
            if (emissionsWeeklyPercentTimes1000 < 1000)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000 + 250;
            }
        else
            {
            if (emissionsWeeklyPercentTimes1000 > 250)
                emissionsWeeklyPercentTimes1000 = emissionsWeeklyPercentTimes1000 - 250;
            }
71: if (increase)
            {
            if (stakingRewardsPercent < 75)
                stakingRewardsPercent = stakingRewardsPercent + 5;
            }
        else
            {
            if (stakingRewardsPercent > 25)
                stakingRewardsPercent = stakingRewardsPercent - 5;
            }
88: if (increase)
            {
            if (percentRewardsSaltUSDS < 25)
                percentRewardsSaltUSDS = percentRewardsSaltUSDS + 5;
            }
        else
            {
            if (percentRewardsSaltUSDS > 5)
                percentRewardsSaltUSDS = percentRewardsSaltUSDS - 5;
            }
```
[38](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L38) | [54](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L54) | [71](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L71) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L88)

```solidity
File: src/stable/StableConfig.sol

69: if (increase)
            {
            if (maxRewardValueForCallingLiquidation < 1000 ether)
                maxRewardValueForCallingLiquidation += 100 ether;
            }
        else
            {
            if (maxRewardValueForCallingLiquidation > 100 ether)
                maxRewardValueForCallingLiquidation -= 100 ether;
            }
86: if (increase)
            {
            if (minimumCollateralValueForBorrowing < 5000 ether)
                minimumCollateralValueForBorrowing += 500 ether;
            }
        else
            {
            if (minimumCollateralValueForBorrowing > 1000 ether)
                minimumCollateralValueForBorrowing -= 500 ether;
            }
103: if (increase)
            {
            if (initialCollateralRatioPercent < 300)
                initialCollateralRatioPercent += 25;
            }
        else
            {
            if (initialCollateralRatioPercent > 150)
                initialCollateralRatioPercent -= 25;
            }
140: if (increase)
            {
            if (percentArbitrageProfitsForStablePOL < 10)
                percentArbitrageProfitsForStablePOL += 1;
            }
        else
            {
            if (percentArbitrageProfitsForStablePOL > 1)
                percentArbitrageProfitsForStablePOL -= 1;
            }
```
[69](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L69) | [86](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L86) | [103](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L103) | [140](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L140)

```solidity
File: src/staking/StakingConfig.sol

36: if (increase)
            {
            if (minUnstakeWeeks < 12)
                minUnstakeWeeks += 1;
            }
        else
            {
            if (minUnstakeWeeks > 1)
                minUnstakeWeeks -= 1;
            }
53: if (increase)
            {
            if (maxUnstakeWeeks < 108)
                maxUnstakeWeeks += 8;
            }
        else
            {
            if (maxUnstakeWeeks > 20)
                maxUnstakeWeeks -= 8;
            }
70: if (increase)
            {
            if (minUnstakePercent < 50)
                minUnstakePercent += 5;
            }
        else
            {
            if (minUnstakePercent > 10)
                minUnstakePercent -= 5;
            }
87: if (increase)
            {
            if (modificationCooldown < 6 hours)
                modificationCooldown += 15 minutes;
            }
        else
            {
            if (modificationCooldown > 15 minutes)
                modificationCooldown -= 15 minutes;
            }
```
[36](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L36) | [53](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L53) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L70) | [87](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L87)

```solidity
File: src/staking/StakingRewards.sol

299: if ( block.timestamp >= cooldownExpiration )
				cooldowns[i] = 0;
			else
				cooldowns[i] = cooldownExpiration - block.timestamp;
```
[299](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L299)
</details>



### [N-04] Avoid Empty Blocks in Code

Empty blocks in code can lead to confusion and the introduction of errors when the code is later modified.
They should be removed, or the block should do something useful, such as emitting an event or reverting. 

For contracts meant to be extended, the contract should be abstract and the function signatures added without any default implementation. 

<details>
<summary><i>14 issue instances in 3 files:</i></summary>

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

50: catch (bytes memory) // Catching any failure
			{
			// In case of failure, price will remain 0
			}
```
[50](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L50)

```solidity
File: src/price_feed/PriceAggregator.sol

155: catch (bytes memory)
			{
			// price remains 0
			}
168: catch (bytes memory)
			{
			// price remains 0
			}
```
[155](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L155) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L168)

```solidity
File: src/Upkeep.sol

247: try this.step1() {}
250: try this.step2(msg.sender) {}
253: try this.step3() {}
256: try this.step4() {}
259: try this.step5() {}
262: try this.step6() {}
265: try this.step7() {}
268: try this.step8() {}
271: try this.step9() {}
274: try this.step10() {}
277: try this.step11() {}
```
[247](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L247) | [250](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L250) | [253](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L253) | [256](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L256) | [259](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L259) | [262](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L262) | [265](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L265) | [268](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L268) | [271](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L271) | [274](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L274) | [277](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L277)
</details>



### [N-05] Consider using OpenZeppelin's SafeCast for any casting

OpenZeppelin's has `SafeCast` library that provides functions to safely cast. Recommended to use it instead of casting directly in any case.

<details>
<summary><i>32 issue instances in 7 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

72: int256(amountOut)
72: int256(midpoint)
75: int256(PoolUtils.DUST)
86: int256(amountOut)
86: int256(midpoint)
133: int256(amountOut)
133: int256(bestArbAmountIn)
133: int256(PoolUtils.DUST)
```
[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L72) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L72) | [75](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L75) | [86](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L86) | [86](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L86) | [133](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L133) | [133](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L133) | [133](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L133)

```solidity
File: src/pools/PoolUtils.sol

24: uint160(address(tokenB))
24: uint160(address(tokenA))
35: uint160(address(tokenB))
35: uint160(address(tokenA))
```
[24](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L24) | [24](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L24) | [35](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L35) | [35](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L35)

```solidity
File: src/pools/PoolStats.sol

68: uint64(i)
```
[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L68)

```solidity
File: src/pools/Pools.sol

100: uint128(maxAmount0)
101: uint128(maxAmount1)
125: uint128(addedAmount0)
126: uint128(addedAmount1)
182: uint128(reclaimedA)
183: uint128(reclaimedB)
274: uint128(reserve0)
275: uint128(reserve1)
```
[100](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L100) | [101](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L101) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L126) | [182](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L182) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L183) | [274](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L274) | [275](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L275)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

59: uint256(price)
```
[59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L59)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

53: uint32(twapInterval)
59: int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)))
59: int56(uint56(twapInterval))
59: uint56(twapInterval)
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L53) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59)

```solidity
File: src/staking/StakingRewards.sol

83: uint128(virtualRewardsToAdd)
84: uint128(virtualRewardsToAdd)
88: uint128(increaseShareAmount)
125: uint128(decreaseShareAmount)
126: uint128(virtualRewardsToRemove)
159: uint128(pendingRewards)
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L84) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L88) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L126) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L159)
</details>



### [N-06] Leverage Recent Solidity Features with `0.8.23`

The recent updates in Solidity provide several features and optimizations that, when leveraged appropriately, can significantly improve your contract's code clarity and maintainability.
Key enhancements include the use of push0 for placing 0 on the stack for EVM versions starting from "Shanghai", making your code simpler and more straightforward.
Moreover, Solidity has extended NatSpec documentation support to enum and struct definitions, facilitating more comprehensive and insightful code documentation.

Additionally, the re-implementation of the UnusedAssignEliminator and UnusedStoreEliminator in the Solidity optimizer provides the ability to remove unused assignments in deeply nested loops.
This results in a cleaner, more efficient contract code, reducing clutter and potential points of confusion during code review or debugging.
It's recommended to make full use of these features and optimizations to enhance the robustness and readability of your smart contracts.

<details>
<summary><i>35 issue instances in 35 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L2)

```solidity
File: src/dao/DAO.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L2)

```solidity
File: src/dao/DAOConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L2)

```solidity
File: src/dao/Proposals.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L2)

```solidity
File: src/dao/Parameters.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L2)

```solidity
File: src/launch/Airdrop.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L2)

```solidity
File: src/launch/BootstrapBallot.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L2)

```solidity
File: src/launch/InitialDistribution.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L2)

```solidity
File: src/pools/PoolUtils.sol

1: pragma solidity =0.8.22;
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L1)

```solidity
File: src/pools/PoolsConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L2)

```solidity
File: src/pools/PoolStats.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L2)

```solidity
File: src/pools/Pools.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L2)

```solidity
File: src/pools/PoolMath.sol

1: pragma solidity =0.8.22;
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L1)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L2)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L2)

```solidity
File: src/price_feed/PriceAggregator.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L2)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L2)

```solidity
File: src/rewards/RewardsConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L2)

```solidity
File: src/rewards/Emissions.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L2)

```solidity
File: src/rewards/RewardsEmitter.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L2)

```solidity
File: src/rewards/SaltRewards.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L2)

```solidity
File: src/stable/USDS.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L2)

```solidity
File: src/stable/StableConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L2)

```solidity
File: src/stable/CollateralAndLiquidity.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L2)

```solidity
File: src/stable/Liquidizer.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L2)

```solidity
File: src/staking/StakingConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L2)

```solidity
File: src/staking/Liquidity.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L2)

```solidity
File: src/staking/StakingRewards.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L2)

```solidity
File: src/staking/Staking.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L2)

```solidity
File: src/ManagedWallet.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L2)

```solidity
File: src/AccessManager.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L2)

```solidity
File: src/Salt.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L2)

```solidity
File: src/SigningTools.sol

1: pragma solidity =0.8.22;
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L1)

```solidity
File: src/Upkeep.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L2)

```solidity
File: src/ExchangeConfig.sol

2: pragma solidity =0.8.22;
```
[2](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L2)
</details>



### [N-07] Mixed usage of `int/uint` with `int256/uint256`

In Solidity, int256 and uint256 are the preferred type names, especially since they are used for function signatures. Using int or uint instead can lead to confusion and inconsistency.
Suggest replacing them with int256 or uint256 respectively, for better clarity and uniformity in your codebase. 
Improving this aspect of your code can contribute to its readability, understandability, and thus maintainability in the long term.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/pools/Pools.sol

81: modifier ensureNotExpired(uint deadline)
```
[81](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L81)

```solidity
File: src/staking/Liquidity.sol

40: modifier ensureNotExpired(uint deadline)
```
[40](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L40)
</details>



### [N-08] Inconsistent spacing in comments

Some lines use // x and some use //x. The instances below point out the usages that don't follow the majority, within each file:

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/pools/PoolMath.sol

187: //        uint256 C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
188: //        uint256 discriminant = B * B - 4 * A * C;
```
[187](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L187) | [188](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L188)
</details>



### [N-09] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>29 issue instances in 24 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

17: constructor( IExchangeConfig _exchangeConfig )
    	{
```
[17](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L17)

```solidity
File: src/dao/DAO.sol

68: constructor( IPools _pools, IProposals _proposals, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IStakingConfig _stakingConfig, IRewardsConfig _rewardsConfig, IStableConfig _stableConfig, IDAOConfig _daoConfig, IPriceAggregator _priceAggregator, IRewardsEmitter _liquidityRewardsEmitter, ICollateralAndLiquidity _collateralAndLiquidity )
		{
```
[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L68)

```solidity
File: src/dao/Proposals.sol

68: constructor( IStaking _staking, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IDAOConfig _daoConfig )
		{
```
[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L68)

```solidity
File: src/launch/Airdrop.sol

33: constructor( IExchangeConfig _exchangeConfig, IStaking _staking )
		{
```
[33](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L33)

```solidity
File: src/launch/BootstrapBallot.sol

31: constructor( IExchangeConfig _exchangeConfig, IAirdrop _airdrop, uint256 ballotDuration )
		{
```
[31](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L31)

```solidity
File: src/launch/InitialDistribution.sol

32: constructor( ISalt _salt, IPoolsConfig _poolsConfig, IEmissions _emissions, IBootstrapBallot _bootstrapBallot, IDAO _dao, VestingWallet _daoVestingWallet, VestingWallet _teamVestingWallet, IAirdrop _airdrop, ISaltRewards _saltRewards, ICollateralAndLiquidity _collateralAndLiquidity  )
		{
```
[32](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L32)

```solidity
File: src/pools/PoolStats.sol

104: function _calculateArbitrageProfits( bytes32[] memory poolIDs, uint256[] memory _calculatedProfits ) internal view
		{
25: constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig )
    	{
```
[104](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L104) | [25](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L25)

```solidity
File: src/pools/Pools.sol

60: function setContracts( IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
		{
```
[60](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L60)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

17: constructor( address _CHAINLINK_BTC_USD, address _CHAINLINK_ETH_USD )
		{
```
[17](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L17)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

30: constructor( IERC20 _wbtc, IERC20 _weth, IERC20 _usdc, address _UNISWAP_V3_WBTC_WETH, address _UNISWAP_V3_WETH_USDC )
		{
```
[30](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L30)

```solidity
File: src/price_feed/PriceAggregator.sol

35: function setInitialFeeds( IPriceFeed _priceFeed1, IPriceFeed _priceFeed2, IPriceFeed _priceFeed3 ) public onlyOwner
		{
```
[35](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L35)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

20: constructor( IPools _pools, IExchangeConfig _exchangeConfig )
		{
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L20)

```solidity
File: src/rewards/Emissions.sol

25: constructor( ISaltRewards _saltRewards, IExchangeConfig _exchangeConfig, IRewardsConfig _rewardsConfig )
		{
```
[25](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L25)

```solidity
File: src/rewards/RewardsEmitter.sol

36: constructor( IStakingRewards _stakingRewards, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IRewardsConfig _rewardsConfig, bool _isForCollateralAndLiquidity )
		{
```
[36](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L36)

```solidity
File: src/rewards/SaltRewards.sol

26: constructor( IRewardsEmitter _stakingRewardsEmitter, IRewardsEmitter _liquidityRewardsEmitter, IExchangeConfig _exchangeConfig, IRewardsConfig _rewardsConfig )
		{
```
[26](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L26)

```solidity
File: src/stable/USDS.sol

29: function setCollateralAndLiquidity( ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
		{
```
[29](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L29)

```solidity
File: src/stable/CollateralAndLiquidity.sol

50: constructor( IPools _pools, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IStakingConfig _stakingConfig, IStableConfig _stableConfig, IPriceAggregator _priceAggregator, ILiquidizer _liquidizer )
		Liquidity( _pools, _exchangeConfig, _poolsConfig, _stakingConfig )
    	{
```
[50](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L50)

```solidity
File: src/stable/Liquidizer.sol

63: function setContracts( ICollateralAndLiquidity _collateralAndLiquidity, IPools _pools, IDAO _dao) external onlyOwner
		{
47: constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig  )
		{
```
[63](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L63) | [47](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L47)

```solidity
File: src/staking/Liquidity.sol

29: constructor( IPools _pools, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IStakingConfig _stakingConfig )
		StakingRewards( _exchangeConfig, _poolsConfig, _stakingConfig )
		{
```
[29](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L29)

```solidity
File: src/staking/StakingRewards.sol

46: constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IStakingConfig _stakingConfig )
		{
```
[46](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L46)

```solidity
File: src/ManagedWallet.sol

42: function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
		{
29: constructor( address _mainWallet, address _confirmationWallet)
		{
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L42) | [29](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L29)

```solidity
File: src/AccessManager.sol

30: constructor( IDAO _dao )
		{
```
[30](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L30)

```solidity
File: src/Upkeep.sol

65: constructor( IPools _pools, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IDAOConfig _daoConfig, IStableConfig _stableConfig, IPriceAggregator _priceAggregator, ISaltRewards _saltRewards, ICollateralAndLiquidity _collateralAndLiquidity, IEmissions _emissions, IDAO _dao )
		{
```
[65](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L65)

```solidity
File: src/ExchangeConfig.sol

48: function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
		{
60: function setAccessManager( IAccessManager _accessManager ) external onlyOwner
		{
34: constructor( ISalt _salt, IERC20 _wbtc, IERC20 _weth, IERC20 _dai, IUSDS _usds, IManagedWallet _managedTeamWallet )
		{
```
[48](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L48) | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L60) | [34](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L34)
</details>



### [N-10] Insufficient Comments in Complex Functions

Complex functions spanning multiple lines should have more comments to better explain the purpose of each logic step.
Proper commenting can greatly improve the readability and maintainability of the codebase. Consider adding explicit comments
to provide insights into the function's operation.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: src/dao/DAO.sol

/// @audit function with 38 lines has only 6 comment lines
153: function _executeApproval( Ballot memory ballot ) internal
		{
```
[153](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L153)

```solidity
File: src/dao/Parameters.sol

/// @audit function with 52 lines has only 6 comment lines
57: function _executeParameterChange( ParameterTypes parameterType, bool increase, IPoolsConfig poolsConfig, IStakingConfig stakingConfig, IRewardsConfig rewardsConfig, IStableConfig stableConfig, IDAOConfig daoConfig, IPriceAggregator priceAggregator ) internal
		{
```
[57](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L57)

```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit function with 31 lines has only 11 comment lines
82: function performUpkeep( uint256 timeSinceLastUpkeep ) external
		{
```
[82](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L82)
</details>



### [N-11] Variable Names Not in mixedCase

State or Local Variable names in your contract don't align with the Solidity naming convention.
For clarity and code consistency, it's recommended to use mixedCase for local and state variables that are not constants, and add a trailing underscore for internal variables.
Adhering to this convention helps in improving code readability and maintenance.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-names)

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: src/price_feed/CoreUniswapFeed.sol

/// @audit - Variable name `uniswapWBTC_WETH` should be in mixedCase
99: uint256 uniswapWBTC_WETH = getUniswapTwapWei( UNISWAP_V3_WBTC_WETH, twapInterval );
/// @audit - Variable name `uniswapWETH_USDC` should be in mixedCase
100: uint256 uniswapWETH_USDC = getUniswapTwapWei( UNISWAP_V3_WETH_USDC, twapInterval );
/// @audit - Variable name `uniswapWETH_USDC` should be in mixedCase
119: uint256 uniswapWETH_USDC = getUniswapTwapWei( UNISWAP_V3_WETH_USDC, twapInterval );
```
[99](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L99) | [100](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L100) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L119)
</details>



### [N-12] Consider making contracts `Upgradeable`

Contract uses a non-upgradeable design.
Transitioning to an upgradeable contract structure is more aligned with contemporary smart contract practices.
This approach not only enhances flexibility but also allows for continuous improvement and adaptation, ensuring the contract stays relevant and robust in an ever-evolving ecosystem.

<details>
<summary><i>32 issue instances in 32 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

/// @audit - Contract `ArbitrageSearch` is not upgradeable
9: abstract contract ArbitrageSearch
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L9)

```solidity
File: src/dao/DAO.sol

/// @audit - Contract `DAO` is not upgradeable
23: contract DAO is IDAO, Parameters, ReentrancyGuard
    {
```
[23](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L23)

```solidity
File: src/dao/DAOConfig.sol

/// @audit - Contract `DAOConfig` is not upgradeable
9: contract DAOConfig is IDAOConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L9)

```solidity
File: src/dao/Proposals.sol

/// @audit - Contract `Proposals` is not upgradeable
20: contract Proposals is IProposals, ReentrancyGuard
    {
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L20)

```solidity
File: src/dao/Parameters.sol

/// @audit - Contract `Parameters` is not upgradeable
10: abstract contract Parameters
    {
```
[10](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L10)

```solidity
File: src/launch/Airdrop.sol

/// @audit - Contract `Airdrop` is not upgradeable
13: contract Airdrop is IAirdrop, ReentrancyGuard
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L13)

```solidity
File: src/launch/BootstrapBallot.sol

/// @audit - Contract `BootstrapBallot` is not upgradeable
12: contract BootstrapBallot is IBootstrapBallot, ReentrancyGuard
    {
```
[12](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L12)

```solidity
File: src/launch/InitialDistribution.sol

/// @audit - Contract `InitialDistribution` is not upgradeable
13: contract InitialDistribution is IInitialDistribution
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L13)

```solidity
File: src/pools/PoolsConfig.sol

/// @audit - Contract `PoolsConfig` is not upgradeable
11: contract PoolsConfig is IPoolsConfig, Ownable
    {
```
[11](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L11)

```solidity
File: src/pools/PoolStats.sol

/// @audit - Contract `PoolStats` is not upgradeable
12: abstract contract PoolStats is IPoolStats
	{
```
[12](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L12)

```solidity
File: src/pools/Pools.sol

/// @audit - Contract `Pools` is not upgradeable
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
```
[22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

/// @audit - Contract `CoreChainlinkFeed` is not upgradeable
10: contract CoreChainlinkFeed is IPriceFeed
    {
```
[10](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L10)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

/// @audit - Contract `CoreUniswapFeed` is not upgradeable
14: contract CoreUniswapFeed is IPriceFeed
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L14)

```solidity
File: src/price_feed/PriceAggregator.sol

/// @audit - Contract `PriceAggregator` is not upgradeable
13: contract PriceAggregator is IPriceAggregator, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L13)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

/// @audit - Contract `CoreSaltyFeed` is not upgradeable
13: contract CoreSaltyFeed is IPriceFeed
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L13)

```solidity
File: src/rewards/RewardsConfig.sol

/// @audit - Contract `RewardsConfig` is not upgradeable
9: contract RewardsConfig is IRewardsConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L9)

```solidity
File: src/rewards/Emissions.sol

/// @audit - Contract `Emissions` is not upgradeable
14: contract Emissions is IEmissions
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L14)

```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit - Contract `RewardsEmitter` is not upgradeable
20: contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
    {
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L20)

```solidity
File: src/rewards/SaltRewards.sol

/// @audit - Contract `SaltRewards` is not upgradeable
15: contract SaltRewards is ISaltRewards
    {
```
[15](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L15)

```solidity
File: src/stable/USDS.sol

/// @audit - Contract `USDS` is not upgradeable
13: contract USDS is ERC20, IUSDS, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L13)

```solidity
File: src/stable/StableConfig.sol

/// @audit - Contract `StableConfig` is not upgradeable
9: contract StableConfig is IStableConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L9)

```solidity
File: src/stable/CollateralAndLiquidity.sol

/// @audit - Contract `CollateralAndLiquidity` is not upgradeable
21: contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
    {
```
[21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L21)

```solidity
File: src/stable/Liquidizer.sol

/// @audit - Contract `Liquidizer` is not upgradeable
21: contract Liquidizer is ILiquidizer, Ownable
    {
```
[21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L21)

```solidity
File: src/staking/StakingConfig.sol

/// @audit - Contract `StakingConfig` is not upgradeable
9: contract StakingConfig is IStakingConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L9)

```solidity
File: src/staking/Liquidity.sol

/// @audit - Contract `Liquidity` is not upgradeable
17: abstract contract Liquidity is ILiquidity, StakingRewards
    {
```
[17](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L17)

```solidity
File: src/staking/StakingRewards.sol

/// @audit - Contract `StakingRewards` is not upgradeable
20: abstract contract StakingRewards is IStakingRewards, ReentrancyGuard
    {
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L20)

```solidity
File: src/staking/Staking.sol

/// @audit - Contract `Staking` is not upgradeable
14: contract Staking is IStaking, StakingRewards
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L14)

```solidity
File: src/ManagedWallet.sol

/// @audit - Contract `ManagedWallet` is not upgradeable
11: contract ManagedWallet is IManagedWallet
    {
```
[11](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L11)

```solidity
File: src/AccessManager.sol

/// @audit - Contract `AccessManager` is not upgradeable
18: contract AccessManager is IAccessManager
	{
```
[18](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L18)

```solidity
File: src/Salt.sol

/// @audit - Contract `Salt` is not upgradeable
6: contract Salt is ISalt, ERC20
    {
```
[6](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L6)

```solidity
File: src/Upkeep.sol

/// @audit - Contract `Upkeep` is not upgradeable
38: contract Upkeep is IUpkeep, ReentrancyGuard
    {
```
[38](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L38)

```solidity
File: src/ExchangeConfig.sol

/// @audit - Contract `ExchangeConfig` is not upgradeable
13: contract ExchangeConfig is IExchangeConfig, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L13)
</details>



### [N-13] Named Imports of Parent Contracts Are Missing

It's important to have named imports for parent contracts to ensure code readability and maintainability.
Missing named imports can make it difficult to understand the code hierarchy and can lead to issues in the future.

<details>
<summary><i>50 issue instances in 27 files:</i></summary>

```solidity
File: src/dao/DAO.sol

/// @audit Missing named import for parent contract: `IDAO`
23: contract DAO is IDAO, Parameters, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `Parameters`
23: contract DAO is IDAO, Parameters, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `ReentrancyGuard`
23: contract DAO is IDAO, Parameters, ReentrancyGuard
    {
```
[23](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L23) | [23](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L23) | [23](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L23)

```solidity
File: src/dao/DAOConfig.sol

/// @audit Missing named import for parent contract: `IDAOConfig`
9: contract DAOConfig is IDAOConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
9: contract DAOConfig is IDAOConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L9) | [9](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L9)

```solidity
File: src/dao/Proposals.sol

/// @audit Missing named import for parent contract: `IProposals`
20: contract Proposals is IProposals, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `ReentrancyGuard`
20: contract Proposals is IProposals, ReentrancyGuard
    {
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L20) | [20](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L20)

```solidity
File: src/launch/Airdrop.sol

/// @audit Missing named import for parent contract: `IAirdrop`
13: contract Airdrop is IAirdrop, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `ReentrancyGuard`
13: contract Airdrop is IAirdrop, ReentrancyGuard
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L13) | [13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L13)

```solidity
File: src/launch/BootstrapBallot.sol

/// @audit Missing named import for parent contract: `IBootstrapBallot`
12: contract BootstrapBallot is IBootstrapBallot, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `ReentrancyGuard`
12: contract BootstrapBallot is IBootstrapBallot, ReentrancyGuard
    {
```
[12](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L12) | [12](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L12)

```solidity
File: src/launch/InitialDistribution.sol

/// @audit Missing named import for parent contract: `IInitialDistribution`
13: contract InitialDistribution is IInitialDistribution
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L13)

```solidity
File: src/pools/PoolsConfig.sol

/// @audit Missing named import for parent contract: `IPoolsConfig`
11: contract PoolsConfig is IPoolsConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
11: contract PoolsConfig is IPoolsConfig, Ownable
    {
```
[11](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L11) | [11](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L11)

```solidity
File: src/pools/Pools.sol

/// @audit Missing named import for parent contract: `IPools`
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
/// @audit Missing named import for parent contract: `ReentrancyGuard`
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
/// @audit Missing named import for parent contract: `PoolStats`
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
/// @audit Missing named import for parent contract: `ArbitrageSearch`
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
/// @audit Missing named import for parent contract: `Ownable`
22: contract Pools is IPools, ReentrancyGuard, PoolStats, ArbitrageSearch, Ownable
	{
```
[22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22) | [22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22) | [22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22) | [22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22) | [22](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L22)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

/// @audit Missing named import for parent contract: `IPriceFeed`
10: contract CoreChainlinkFeed is IPriceFeed
    {
```
[10](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L10)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

/// @audit Missing named import for parent contract: `IPriceFeed`
14: contract CoreUniswapFeed is IPriceFeed
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L14)

```solidity
File: src/price_feed/PriceAggregator.sol

/// @audit Missing named import for parent contract: `IPriceAggregator`
13: contract PriceAggregator is IPriceAggregator, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
13: contract PriceAggregator is IPriceAggregator, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L13) | [13](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L13)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

/// @audit Missing named import for parent contract: `IPriceFeed`
13: contract CoreSaltyFeed is IPriceFeed
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L13)

```solidity
File: src/rewards/RewardsConfig.sol

/// @audit Missing named import for parent contract: `IRewardsConfig`
9: contract RewardsConfig is IRewardsConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
9: contract RewardsConfig is IRewardsConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L9) | [9](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L9)

```solidity
File: src/rewards/Emissions.sol

/// @audit Missing named import for parent contract: `IEmissions`
14: contract Emissions is IEmissions
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L14)

```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit Missing named import for parent contract: `IRewardsEmitter`
20: contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
    {
/// @audit Missing named import for parent contract: `ReentrancyGuard`
20: contract RewardsEmitter is IRewardsEmitter, ReentrancyGuard
    {
```
[20](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L20) | [20](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L20)

```solidity
File: src/rewards/SaltRewards.sol

/// @audit Missing named import for parent contract: `ISaltRewards`
15: contract SaltRewards is ISaltRewards
    {
```
[15](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L15)

```solidity
File: src/stable/USDS.sol

/// @audit Missing named import for parent contract: `ERC20`
13: contract USDS is ERC20, IUSDS, Ownable
    {
/// @audit Missing named import for parent contract: `IUSDS`
13: contract USDS is ERC20, IUSDS, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
13: contract USDS is ERC20, IUSDS, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L13) | [13](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L13) | [13](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L13)

```solidity
File: src/stable/StableConfig.sol

/// @audit Missing named import for parent contract: `IStableConfig`
9: contract StableConfig is IStableConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
9: contract StableConfig is IStableConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L9) | [9](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L9)

```solidity
File: src/stable/CollateralAndLiquidity.sol

/// @audit Missing named import for parent contract: `Liquidity`
21: contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
    {
/// @audit Missing named import for parent contract: `ICollateralAndLiquidity`
21: contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
    {
```
[21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L21) | [21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L21)

```solidity
File: src/stable/Liquidizer.sol

/// @audit Missing named import for parent contract: `ILiquidizer`
21: contract Liquidizer is ILiquidizer, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
21: contract Liquidizer is ILiquidizer, Ownable
    {
```
[21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L21) | [21](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L21)

```solidity
File: src/staking/StakingConfig.sol

/// @audit Missing named import for parent contract: `IStakingConfig`
9: contract StakingConfig is IStakingConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
9: contract StakingConfig is IStakingConfig, Ownable
    {
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L9) | [9](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L9)

```solidity
File: src/staking/Staking.sol

/// @audit Missing named import for parent contract: `IStaking`
14: contract Staking is IStaking, StakingRewards
    {
/// @audit Missing named import for parent contract: `StakingRewards`
14: contract Staking is IStaking, StakingRewards
    {
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L14) | [14](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L14)

```solidity
File: src/ManagedWallet.sol

/// @audit Missing named import for parent contract: `IManagedWallet`
11: contract ManagedWallet is IManagedWallet
    {
```
[11](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L11)

```solidity
File: src/AccessManager.sol

/// @audit Missing named import for parent contract: `IAccessManager`
18: contract AccessManager is IAccessManager
	{
```
[18](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L18)

```solidity
File: src/Salt.sol

/// @audit Missing named import for parent contract: `ISalt`
6: contract Salt is ISalt, ERC20
    {
/// @audit Missing named import for parent contract: `ERC20`
6: contract Salt is ISalt, ERC20
    {
```
[6](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L6) | [6](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L6)

```solidity
File: src/Upkeep.sol

/// @audit Missing named import for parent contract: `ReentrancyGuard`
38: contract Upkeep is IUpkeep, ReentrancyGuard
    {
```
[38](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L38)

```solidity
File: src/ExchangeConfig.sol

/// @audit Missing named import for parent contract: `IExchangeConfig`
13: contract ExchangeConfig is IExchangeConfig, Ownable
    {
/// @audit Missing named import for parent contract: `Ownable`
13: contract ExchangeConfig is IExchangeConfig, Ownable
    {
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L13) | [13](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L13)
</details>



### [N-14] Long functions should be refactored into multiple, smaller, functions

Functions that span many lines can be hard to understand and maintain.
It is often beneficial to refactor long functions into multiple smaller functions.
This improves readability, makes testing easier, and can even lead to gas optimization if it eliminates the need for variables that would otherwise have to be stored in memory.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: src/dao/DAO.sol

 /// @audit 38 lines (excluding comments)
153: function _executeApproval( Ballot memory ballot ) internal
		{
```
[153](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L153)

```solidity
File: src/dao/Parameters.sol

 /// @audit 52 lines (excluding comments)
57: function _executeParameterChange( ParameterTypes parameterType, bool increase, IPoolsConfig poolsConfig, IStakingConfig stakingConfig, IRewardsConfig rewardsConfig, IStableConfig stableConfig, IDAOConfig daoConfig, IPriceAggregator priceAggregator ) internal
		{
```
[57](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L57)

```solidity
File: src/rewards/RewardsEmitter.sol

 /// @audit 31 lines (excluding comments)
82: function performUpkeep( uint256 timeSinceLastUpkeep ) external
		{
```
[82](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L82)
</details>



### [N-15] Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments

When using `abi.encodePacked()` on a single argument, it is often clearer to use a cast to `bytes` or `bytes32`.
This improves the semantic clarity of the code, making it easier for reviewers to understand the developer's intentions.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/dao/Proposals.sol

251: require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
251: require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
```
[251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251) | [251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251)
</details>



### [N-16] Use Unchecked for Divisions on Constant or Immutable Values

Unsigned divisions on constant or immutable values do not result in overflow.
Therefore, these operations can be marked as unchecked, optimizing gas usage without compromising safety.

For instance, if `a` is an unsigned integer and `b` is a constant or immutable, a / b can be safely rewritten as: unchecked { a / b }

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/stable/CollateralAndLiquidity.sol

202: uint256 btcValue = ( amountBTC * btcPrice ) / wbtcTenToTheDecimals;
203: uint256 ethValue = ( amountETH * ethPrice ) / wethTenToTheDecimals;
```
[202](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L202) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L203)
</details>
