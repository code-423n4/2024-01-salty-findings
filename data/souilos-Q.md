# INFO - Discrepancy between the doc and the code

## Description

On Salty's website https://docs.salty.io/the-salt-token/emissions in the page that talks about xSALT (rewards), we can read:
"... a default of 45% of the SALT rewards that are acquired by the DAO through Protocol Owned Liquidity." 

While in the code https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L28 this is set to 50%:

```
	// For rewards distributed to the DAO, the percentage of SALT that is burned with the remaining staying in the DAO for later use.
	// Range: 25% to 75% with an adjustment of 5%
    uint256 public percentPolRewardsBurned = 50;
```

Moreover, as you can see in the comment, this `percentPolRewardsBurned` must be able to be adjusted by 5% from 25 to 75.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L81

```solidity
	function changePercentPolRewardsBurned(bool increase) external onlyOwner
		{
		if (increase)
			{
			if (percentPolRewardsBurned < 75)
				percentPolRewardsBurned += 5;
			}
		else
			{
			if (percentPolRewardsBurned > 25)
				percentPolRewardsBurned -= 5;
			}

		emit PercentPolRewardsBurnedChanged(percentPolRewardsBurned);
		}
```

However the dev and owner of the protocol said on Discord something different:
From me:
souilos — 19/01/2024 10:19
Hi @Daniel Cota, 
"A default 45% of the SALT rewards received from this POL are burned". 
I am just wondering why 45% ? 🙂

Him:
Daniel Cota — 19/01/2024 10:28
Seemed like a reasonable starting point. The DAO can adjust the percent between 23% and 68%.

## Mitigation

This is just fully informational, but it's important to make sure the doc is aligned with the code.