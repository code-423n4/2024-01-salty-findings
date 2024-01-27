# Possible reentrancy attack and cooldown manipulation
Instances:
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L57-L92
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L97-L140

## PoC

Absence of reentrancy guards in the functions `_increaseUserShare` and `_decreaseUserShare` 
and possibly faulty cooldown mechanism:
```
if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
	{
@>	require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

	// Update the cooldown expiration for future transactions
@>	user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
	}
```
If we check interface `IStakingConfig.sol` we see that function `modificationCooldown` doesn't provide much functionality, seems like it's handled manually, by admins. This can lead to man-made risks.

There is a function below `userCooldowns`, but it seems to have nothing to do with the function above. 
Probably, user cooldown mechanism should be reviewed.

Even though functions `_increaseUserShare` and `_decreaseUserShare` are internal, they handle funds, 
so the reentrancy guards should contribute a lot for the function security. 


## Impact
Loss of funds

## Tools:
VSCode

## Recommeded mitigation steps:
1. Consider adding reentrancy guards for the functions `_increaseUserShare` and `_decreaseUserShare`.
2. Consider making strinct comparison , just for case if `stakingConfig.modificationCooldown()` will be equal to 0:
```
function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
	{
	require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
	require( increaseShareAmount != 0, "Cannot increase zero share" );

	UserShareInfo storage user = _userShareInfo[wallet][poolID];

	if ( useCooldown )
	if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
		{
-		require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
+		require( block.timestamp > user.cooldownExpiration, "Must wait for the cooldown to expire" );
```
3. Consider making cooldown mechanism automated, not manual. It can be coded as constant and calculated with function afterwards.
