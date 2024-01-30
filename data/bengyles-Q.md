# QA report

## Function that modifies the state has no access control

### lines

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L77>

### Impact

Contracts inheriting from this abstract contract have a public function available which changes the state and can be called by anyone. There are no parameters for this function that can be exploited but there should be no reason for a user to call this function as the indices only have to be updated after adding a token to the whitelist or removing a token from the whitelist.

### Proof of Concept

Add the following test to `src/pools/tests/Pools.t.sol`

```javascript
  function testUpdateArbitrageIndicies(address user) public {
        // anyone can call this function
        vm.prank(user);
        pools.updateArbitrageIndicies();
    }
```

### Tools Used

vsCodium, Foundry

### Recommended Mitigation Steps

add the onlyOwner modifier to the function, which might need to be imported

```diff
+ function updateArbitrageIndicies() public onlyOwner {
        bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

        for (uint256 i = 0; i < poolIDs.length; i++) {
            bytes32 poolID = poolIDs[i];
            (IERC20 arbToken2, IERC20 arbToken3) = poolsConfig
                .underlyingTokenPair(poolID);

            // The middle two tokens can never be WETH in a valid arbitrage path as the path is WETH->arbToken2->arbToken3->WETH.
            if ((arbToken2 != _weth) && (arbToken3 != _weth)) {
                uint64 poolIndex1 = _poolIndex(_weth, arbToken2, poolIDs);
                uint64 poolIndex2 = _poolIndex(arbToken2, arbToken3, poolIDs);
                uint64 poolIndex3 = _poolIndex(arbToken3, _weth, poolIDs);

                // Check if the indicies in storage have the correct values - and if not then update them
                ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
                if (
                    (poolIndex1 != indicies.index1) ||
                    (poolIndex2 != indicies.index2) ||
                    (poolIndex3 != indicies.index3)
                )
                    _arbitrageIndicies[poolID] = ArbitrageIndicies(
                        poolIndex1,
                        poolIndex2,
                        poolIndex3
                    );
            }
        }
    }

```

Or add a custom check for specified contracts or addresses that are allowed to call this function, this is just an example though and depends on which roles contracts the developer thinks should have this permission.

```diff
function updateArbitrageIndicies() public {
+   require(msg.sender == address(exchangeConfig.dao()), "PoolStats.updateArbitrageIndicies is only callable from the dao");
        bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

        for (uint256 i = 0; i < poolIDs.length; i++) {
            bytes32 poolID = poolIDs[i];
            (IERC20 arbToken2, IERC20 arbToken3) = poolsConfig
                .underlyingTokenPair(poolID);

            // The middle two tokens can never be WETH in a valid arbitrage path as the path is WETH->arbToken2->arbToken3->WETH.
            if ((arbToken2 != _weth) && (arbToken3 != _weth)) {
                uint64 poolIndex1 = _poolIndex(_weth, arbToken2, poolIDs);
                uint64 poolIndex2 = _poolIndex(arbToken2, arbToken3, poolIDs);
                uint64 poolIndex3 = _poolIndex(arbToken3, _weth, poolIDs);

                // Check if the indicies in storage have the correct values - and if not then update them
                ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];
                if (
                    (poolIndex1 != indicies.index1) ||
                    (poolIndex2 != indicies.index2) ||
                    (poolIndex3 != indicies.index3)
                )
                    _arbitrageIndicies[poolID] = ArbitrageIndicies(
                        poolIndex1,
                        poolIndex2,
                        poolIndex3
                    );
            }
        }
    }

```

## Some comments have typos

### lines

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L32> => `updatef` should be `updated`

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L142C100-L142C101> => `and` should be `or`

### Impact

There's no impact for this, just thought I should mention it

vsCodium

### Recommended Mitigation Steps

fix the typos
