## Context
The `performUpKeep` inside the `RewardsEmitter` doesn't check inside the loop if the pool that is being iterated through has rewards, which will cause the function to revert because of division by zero. This will make the whole transaction revert when a pool has zero pending rewards. 
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L121
```solidity
uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;
```

## Impact
Pools that have pending rewards to be distributed will get blocked by pools that have zero pending rewards. Thus, the protocol will reward(standard rate 5% of existing WETH arbitrage profits from the Pools) the caller but yet the distribution of rewards will not happen accordingly. The transaction will not revert because the `Upkeep` contract adds all the calls inside try/catch blocks.

This will cause many users to be unfairly rewarded and also reward the caller the full amount even when the reward distribution doesn't happen as expected. As there is no guarantee that all the pools will always have rewards available when upkeep is called, this is likely to happen often.

## PoC
```solidity
		// Rewards to emit = pendingRewards * timeSinceLastUpkeep * rewardsEmitterDailyPercent / oneDay
		uint256 numeratorMult = timeSinceLastUpkeep * rewardsConfig.rewardsEmitterDailyPercentTimes1000();
		uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000

		uint256 sum = 0;
		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];

			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;

			// Reduce the pending rewards so they are not sent again
			if ( amountToAddForPool != 0 )
				{
				pendingRewards[poolID] -= amountToAddForPool;

				sum += amountToAddForPool;
				}

			// Specify the rewards that will be added for the specific pool
			addedRewards[i] = AddedReward( poolID, amountToAddForPool );
			}
```

## Tools Used
Manual Review

## Recommendation
Check if pendingRewards for a specific pool is greater than zero, if that's the case, skip it. 

```diff
for( uint256 i = 0; i < poolIDs.length; i++ )
			{
			bytes32 poolID = poolIDs[i];
+			if (pendingRewards[poolID] == 0) continue;

			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;

```
