### Using immutable variables can save gas

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ExchangeConfig.sol#L24-L31

```javascript
	IDAO public dao; // can only be set once
	IUpkeep public upkeep; // can only be set once
	IInitialDistribution public initialDistribution; // can only be set once
	IAirdrop public airdrop; // can only be set once

	// Gradually distribute SALT to the teamWallet and DAO over 10 years
	VestingWallet public teamVestingWallet;		// can only be set once
	VestingWallet public daoVestingWallet;		// can only be set once

```
Considering that these variables can only be set once, changing them from storage variables to immutable can save gas

```javascript
	function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
		{
		// setContracts is only called once (on deployment)
		require( address(dao) == address(0), "setContracts can only be called once" );

		dao = _dao;
		upkeep = _upkeep;
		initialDistribution = _initialDistribution;
		airdrop = _airdrop;
		teamVestingWallet = _teamVestingWallet;
		daoVestingWallet = _daoVestingWallet;
		}
```

Considering that these variables can only be set once, changing them from storage variables to immutable can save gas