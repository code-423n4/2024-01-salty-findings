https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L185-L187


        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");


The above require statement compares reserves.reserve0 twice to PoolUtils.DUST rather than comparing reserves.reserve1 to PoolUtils.DUST on the right hand side of &&. Comment indicates that intended functionality is to compare both reserves.reserve0 and reserves.reserve1 to PoolUtils.Dust. Simply change to below require statement:

       require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");