# `PoolConfig::whitelistPool` can return PoolID to save gas.

## Description

The `DAO::_finalizeTokenWhitelisting` function whitelists pools using `poolsConfig.whitelistPool`. This function already retrieves the poolID, but it doesn't return it. Modifying `PoolConfig::whitelistPool` to return the poolID can save approximately 1838 gas units per call, resulting in cost savings.

These savings can be significant, especially considering the current gas prices. With an average gas price of 20 gwei per unit and the price of Ether at \$2250, this modification can save more than \$0.08 for each call. It's important to note that `PoolConfig::whitelistPool` is only called in `DAO::_finalizeTokenWhitelisting`, which doesn’t imply a big refactor.

```javascript
    function _finalizeTokenWhitelisting(uint256 ballotID) internal {
        if (proposals.ballotIsApproved(ballotID)) {
            ...
@>            poolsConfig.whitelistPool(
@>                pools,
@>                IERC20(ballot.address1),
@>                exchangeConfig.wbtc()
@>            );
@>            poolsConfig.whitelistPool(
@>                pools,
@>                IERC20(ballot.address1),
@>                exchangeConfig.weth()
@>            );

@>            bytes32 pool1 = PoolUtils._poolID(
@>                IERC20(ballot.address1),
@>                exchangeConfig.wbtc()
@>            );
@>            bytes32 pool2 = PoolUtils._poolID(
@>                IERC20(ballot.address1),
@>               exchangeConfig.weth()
@>            );
            ...
        }
        ...
    }
```

## Impact

Optimizes gas consumption.

## Proof of Concept

### Foundry PoC added to `DAO.t.sol`

```javascript
function testWhitelistTokenApprovedGasEfficiency() public {
    vm.startPrank(alice);
    staking.stakeSALT(1000000 ether);

    IERC20 token = new TestERC20("TEST", 18);

    proposals.proposeTokenWhitelisting(token, "", "");

    uint256 ballotID = 1;
    proposals.castVote(ballotID, Vote.YES);

    // Increase block time to finalize the ballot
    vm.warp(block.timestamp + 11 days);

    // Test Parameter Ballot finalization
    salt.transfer(address(dao), 399999 ether);
    salt.transfer(address(dao), 5 ether);

    uint gas_before = gasleft();
    dao.finalizeBallot(ballotID);
    uint gas_after = gasleft();
    console.log(gas_before - gas_after);
}
```

## Recommended Mitigation

**IPoolsConfig.sol**

```diff
interface IPoolsConfig {
    function whitelistPool(
        IPools pools,
        IERC20 tokenA,
        IERC20 tokenB
-    ) external; // onlyOwner
+    ) external returns (bytes32); // onlyOwner

    ...
}
```

**PoolsConfig.sol**

```diff
function whitelistPool(
    IPools pools,
    IERC20 tokenA,
    IERC20 tokenB
-) external onlyOwner {
+) external onlyOwner returns (bytes32 poolID) {
    require(
        _whitelist.length() < maximumWhitelistedPools,
        "Maximum number of whitelisted pools already reached"
    );
    require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
-   bytes32 poolID = PoolUtils._poolID(tokenA, tokenB);
+    poolID = PoolUtils._poolID(tokenA, tokenB);

    // Add to the whitelist and remember the underlying tokens for the pool
    _whitelist.add(poolID);
    underlyingPoolTokens[poolID] = TokenPair(tokenA, tokenB);

    // Make sure that the cached arbitrage indicies in PoolStats are updated
    pools.updateArbitrageIndicies();

    emit PoolWhitelisted(address(tokenA), address(tokenB));
}
```

**DAO.sol**

```diff
    function _finalizeTokenWhitelisting(uint256 ballotID) internal {
        if (proposals.ballotIsApproved(ballotID)) {
            ...
-            poolsConfig.whitelistPool(
+            bytes32 pool1 = poolsConfig.whitelistPool(
                pools,
                IERC20(ballot.address1),
                exchangeConfig.wbtc()
            );
-            poolsConfig.whitelistPool(
+            bytes32 pool2 = poolsConfig.whitelistPool(
                pools,
                IERC20(ballot.address1),
                exchangeConfig.weth()
            );

-            bytes32 pool1 = PoolUtils._poolID(
-                IERC20(ballot.address1),
-                exchangeConfig.wbtc()
-            );
-            bytes32 pool2 = PoolUtils._poolID(
-                IERC20(ballot.address1),
-               exchangeConfig.weth()
-            );
            ...
        }
        ...
    }
```