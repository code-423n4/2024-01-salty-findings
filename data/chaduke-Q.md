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