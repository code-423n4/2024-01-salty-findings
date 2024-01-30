### Zero address checks

Have zero address checks in initialDistribution.sol, BootstrapBallot.sol,
pool.sol, setContracts

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L60
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L34
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L34

### Alternate mechanism to Bootstrap

By default, if the whitelisted users decided not to start the protocol, all the protocols are re-deployed. A mechanism where the Bootstrap contract can deploy all the necessary contracts can save protocol funds

### Check return values of Enumberable Sets !!!
The return value of Enumberable sets state variables are not checked, It returns a bool on whether an element was really added or removed.
This might cause an edge case that leads to a pricey security issue in the future

some of many cases..
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L105
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L132
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L185

