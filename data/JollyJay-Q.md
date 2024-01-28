 # Constructor written in a wrong way

## Impact
***It is important to follow best practices when writing a constructor in Solidity. However, the constructor in the pool.sol contract is not written properly. On the other hand, a contract in the pools folder has a well-written constructor that will not cause any errors during deployment.***

## Proof of Concept
```
	constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig )
	ArbitrageSearch(_exchangeConfig)
	PoolStats(_exchangeConfig, _poolsConfig)
		{
		}
```
***Look closely at the constructor the AbritrageSearch and PoolsStats come before the curly braces({) which shouldn't be.***

## Tools Used
Manual review

## Recommended Mitigation Steps
https://github.com/code-423n4/2024-01-salty/commit/f6e5b82dd9368daa41d976c0e549d8b73f4f19fa#diff-ac1567b6d6f64912bb3767c9ee5c1991a6555c69fc1d65aa631991cbf9005096