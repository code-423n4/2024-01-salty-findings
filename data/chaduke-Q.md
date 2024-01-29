QA1: functions _increaseUserShares() and _decreaseUserShares() fail to update user.cooldownExpiration when useCooldown = false. As a result, right after _increaseUserShares() is called with useCooldown = false, another call _increaseUserShares() can be possible since user.cooldownExpiration has not been updated in the previous calls.

[https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140)

Mitigation: Regardless of the value of useCooldown, we should always update user.cooldownExpiration as follows:

```javascript
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57-L140
```