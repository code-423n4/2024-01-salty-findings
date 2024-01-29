### Report 1:
#### Incomplete State Boundary Implementation
maximumWhitelistedPools can be set below the current length of whitelisted pools in the PoolsConfig.sol contract this should be corrected as provided below.
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L87
```solidity
function changeMaximumWhitelistedPools(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (maximumWhitelistedPools < 100)
                maximumWhitelistedPools += 10;
            }
        else
            {
            if (maximumWhitelistedPools > 20 )
                maximumWhitelistedPools -= 10;
+++            if( maximumWhitelistedPools <  _whitelist.length() ){
+++              revert("Error Message");
+++             }
            }

		emit MaximumWhitelistedPoolsChanged(maximumWhitelistedPools);
        }
```
###  Report 2:
