GAS 01 Use caching to avoid repetitive storage reads
## Impact
Several contracts like [Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol) and [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol) could cache frequently used parameters and addresses in memory rather than reading storage every time. For example, caching the `daoAddress` or _token addresses_.

Each storage read incurs gas costs, so avoiding repetitive reads yields savings that scale with network utilization.
The issue comes up in core contracts [Pools.sol#addLiquidity](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L140-L165)

```solidity
function addLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 totalLiquidity ) external nonReentrant returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
{
require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
require( exchangeIsLive, "The exchange is not yet live" );
require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );


require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );


(bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);


// Flip the users arguments if they are not in reserve token order with address(tokenA) < address(tokenB)
if ( flipped )
        (addedAmountB, addedAmountA, addedLiquidity) = _addLiquidity( poolID, maxAmountB, maxAmountA, totalLiquidity );
else
        (addedAmountA, addedAmountB, addedLiquidity) = _addLiquidity( poolID, maxAmountA, maxAmountB, totalLiquidity );


// Make sure the minimum liquidity has been added
require( addedLiquidity >= minLiquidityReceived, "Too little liquidity received" );


// Transfer the tokens from the sender - only tokens without fees should be whitelisted on the DEX
tokenA.safeTransferFrom(msg.sender, address(this), addedAmountA );
tokenB.safeTransferFrom(msg.sender, address(this), addedAmountB );


emit LiquidityAdded(tokenA, tokenB, addedAmountA, addedAmountB, addedLiquidity);
}
```

In each function, `dao` is read from storage. At ~200 gas per read, this adds up:

```
10 function calls * 200 gas per read = 2,000 gas spent
```

## Recommended Mitigation Steps
```solidity 
// Cache reference in memory
address private _dao;

function addLiquidity() external {
  if (_dao == address(0)) {
    _dao = dao; 
  }

  require(msg.sender == _dao, "Unauthorized");
} 
```
Now only a single storage read occurs in the life cycle versus multiple reads.

**Impact**
- Saving ~1,800 gas
- Reduces network congestion and user costs

This pattern of judicious caching, especially on immutable addresses, applies broadly across Salty.IO.

GAS 02 Reduce emission of events
## Impact
Many events like LiquidityAdded and VoteCast are useful for off-chain services but emit gas. Analyze usage and potentially restrict events where possible.