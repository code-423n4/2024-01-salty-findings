|no |issue|number of issue|
|------|-----|---------------|
|[G-01]|The setAccessManager function lacks check address(_accessManager)! = address(accessManager)|1|

## [G01] The setAccessManager function lacks check address(_accessManager)! = address(accessManager)

```solidity

62 	function setAccessManager( IAccessManager _accessManager ) external onlyOwner
		{
		require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" );

		accessManager = _accessManager;

	    emit AccessManagerSet(_accessManager);
		}

```
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ExchangeConfig.sol#L62-L69