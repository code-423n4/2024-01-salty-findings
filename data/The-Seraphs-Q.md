## [QA/Low-01] Use `flipped` in `Pools::removeLiquidity()` function so that the users do not have to order the tokens correctly in the function call, similar to how its done for all other functions

### Code references

https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L175

### Description
The function `Pools::removeLiquidity()` retrieves the pool ID and if the token IDs are flipped.

```solidity
(bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);
```

The pool ID is determined as follows:

```solidity
function _poolIDAndFlipped( IERC20 tokenA, IERC20 tokenB ) internal pure returns (bytes32 poolID, bool flipped)
{
    // See if the token orders are flipped
    if ( uint160(address(tokenB)) < uint160(address(tokenA)) )
        return (keccak256(abi.encodePacked(address(tokenB), address(tokenA))), true);

    return (keccak256(abi.encodePacked(address(tokenA), address(tokenB))), false);
}
```

Flipped is returned as true if `tokenB < tokenA`.

This ensures that the `tokenA`, `tokenB` and the corresponding amounts provided in the function call matches the tokens for `reserve0` and `reserve1` of the pool.

However, `flipped` is not used in the function at all, and its the user's responsibility to provide the tokens in the right order.

The other functions like `Pools::addLiquidity()` use `flipped` and flip the tokens if needed.

### Recommendation

Use `flipped` in the code to flip the tokens if the user provides the tokens in the wrong order.

---

## [QA/Low-02] Add checks in `DAO::_executeSetContract()` to ensure the same price feed is not set for different price feeds

### Code references

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L135-L142

### Description

In `DAO::_executeSetContract()`, based on the `nameHash`, the price feed is set to `priceFeedNum`.

```solidity
if ( nameHash == keccak256(bytes( "setContract:priceFeed1_confirm" )) )
    priceAggregator.setPriceFeed( 1, IPriceFeed(ballot.address1) );
else if ( nameHash == keccak256(bytes( "setContract:priceFeed2_confirm" )) )
    priceAggregator.setPriceFeed( 2, IPriceFeed(ballot.address1) );
else if ( nameHash == keccak256(bytes( "setContract:priceFeed3_confirm" )) )
    priceAggregator.setPriceFeed( 3, IPriceFeed(ballot.address1) );
```

There are no checks here that ensure that the price feed is already not set for another `priceFeedNum`, that is, price feed 1, 2 and 3 all can point to the same price feed.

### Recommendation

Add checks to ensure that price feed 1, 2 and 3 are always different.

---

## [QA/Low-03] Use modifiers in `DAO.sol` contract instead of writing the same checks multiple times

### Code references

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L318

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L329

### Description

In `DAO.sol`, there are multiple functions that validate that only Upkeep contract calls the function.

```solidity
require( msg.sender == address(exchangeConfig.upkeep()), "DAO.XYZ is only callable from the Upkeep contract" );
```

This check has been added multiple times in the functions in the contract.

This can be replaced by a `modifier` to make the code more readable, avoid mistakes in case the code needs update, and follow Do Not Repeat(DRY) principles to not repeat the same code multiple times

### Recommendation

Add a modifier that performs the require check, and use them in the functions

---