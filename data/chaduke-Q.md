QA1: functions _increaseUserShares() and _decreaseUserShares() fail to update user.cooldownExpiration when useCooldown = false. As a result, right after _increaseUserShares() is called with useCooldown = false, another call _increaseUserShares() can be possible since user.cooldownExpiration has not been updated in the previous calls.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140)

Mitigation: Regardless of the value of useCooldown, we should always update user.cooldownExpiration as follows:

```javascript
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140
```

QA2. The _decreaseUserShare() function has a rounding-down precision error for ``virtualRewardsToRemove`` that is in favor of the user instead of the protocol.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L118C11-L118C33](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L118C11-L118C33)

As a result, even when a user redeems all shares, user.virtualRewards still might be positive. Due to the rounding down precision of ``virtualRewardsToRemove``, claiming is in favor instead of the system. 

Mitigation: the rounding should be in favor of the system. A rounding up should be used as follows: 

```javascipt
uint256 virtualRewardsToRemove = Math.ceilDiv(user.virtualRewards * decreaseShareAmount), user.userShare);
```

QA3: _sendLiquidityRewards() might not send all ``liquidityRewardsAmount`` to the pools due to rounding down error. This is because when calculating the portion for each pool, there is a rounding down error:  

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L67-L68](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L67-L68)

As a result, when each time performUpKeep() is called, there might be leftover ``liquidityRewardsAmount`` that has not been distributed. 

Mitigation: send the leftover either to the last pool, or better yet, send it to the staking as follows in the function ``performUpKeep()``: 

```
		_sendLiquidityRewards(liquidityRewardsAmount, directRewardsForSaltUSDS, poolIDs, profitsForPools, totalProfits);
_sendStakingRewards(salt.balanceOf(address(this)) );

```

QA4. userCollateralValueInUSD() might underestimate the collateral value of a user. The main problem is that it calculates ``userWBTC`` and ``userWETH``, which have a rounding down error. In particular, for userWBTC, the decimals is 8. Therefore, the maximum of round down error is 0.00000001 BTC, whose value might not be ignorable in the future when the price of BTC increases to a significant high. 

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L221-L236]

Mitigation: calculate the total reserve value first and then calculate the user collateral value based on his shares:

```javascript
function userCollateralValueInUSD( address wallet ) public view returns (uint256)
		{
		uint256 userCollateralAmount = userShareForPool( wallet, collateralPoolID );
		if ( userCollateralAmount == 0 )
			return 0;

		return totalCollateralValueInUSD()  * userCollateralAmount / totalShares[collateralPoolID];

```

QA5: liquidateUser() still return rewards back to the borrower.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L140C11-L188](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L140C11-L188)

Mitigation: maybe the rewards should be kept by the protocol as a penalty for the user to be insolvent. 

QA6: function addLiquidty() checks ``maxAmountA`` and ``maxAmountB`` to make sure they are not too small (> PoolUtils.DUST). However, the real used amount is: ``addedAmountA`` and ``addedAmountB``. Since one of these values will be smaller than the original ``maxAmountA`` and ``maxAmountB``, it is better to check the amounts in ``addedAmountA`` and ``addedAmountB`` than ``maxAmountA`` and ``maxAmountB``.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L140-L165](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L140-L165)

Mitigation: check ``addedAmountB`` than ``maxAmountA`` and make sure they are > PoolUtils.DUST. 

