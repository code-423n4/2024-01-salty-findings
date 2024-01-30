## Links to affected code

https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L121-L140

## Impact

Before calling the `_decreaseUserShare` function to reduce the share of the user in the corresponding poolid, the safeTransfer function is invoked to transfer tokens.

If the token is ERC777 token, there may be a potential read-only reentrancy, once an external system relies on the `userShareForPool` function in the salt project, we can attack the external system in the ERC777 callback function.

Although through investigation, no cross contract and read-only reentrancy has been found in this project so far, the potential factors that lead to re-entry should be fixed.

## Proof of Concept

Here is a possible attack scenario:
1. An ERC777 token passed through the pool whitelist
2. Malicious contract call to the `withdrawLiquidityAndClaim` function, entering the ERC777 callback function
3. An external system calculates something through `userShareForPool`, and this will lead to incorrect results

## Tools Used

Visual Studio Code, Manual Code Review

## Recommended Mitigation Steps

Suggest reducing share before transferring tokens.

```diff
    function _withdrawLiquidityAndClaim( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToWithdraw, uint256 minReclaimedA, uint256 minReclaimedB ) internal returns (uint256 reclaimedA, uint256 reclaimedB)
        {
        bytes32 poolID = PoolUtils._poolID( tokenA, tokenB );
        require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

        // Remove the amount of liquidity specified by the user.
        // The liquidity in the pool is currently owned by this contract. (external call)
        (reclaimedA, reclaimedB) = pools.removeLiquidity( tokenA, tokenB, liquidityToWithdraw, minReclaimedA, minReclaimedB, totalShares[poolID] );
--
--      // Transfer the reclaimed tokens to the user
--      tokenA.safeTransfer( msg.sender, reclaimedA );
--      tokenB.safeTransfer( msg.sender, reclaimedB );

        // Reduce the user's liquidity share for the specified pool so that they receive less rewards.
        // Cooldown is specified to prevent reward hunting (ie - quickly depositing and withdrawing large amounts of liquidity to snipe rewards)
        // This call will send any pending SALT rewards to msg.sender as well.
        _decreaseUserShare( msg.sender, poolID, liquidityToWithdraw, true );
++
++      // Transfer the reclaimed tokens to the user
++      tokenA.safeTransfer( msg.sender, reclaimedA );
++      tokenB.safeTransfer( msg.sender, reclaimedB );
        emit LiquidityWithdrawn(msg.sender, address(tokenA), address(tokenB), reclaimedA, reclaimedB, liquidityToWithdraw);
        }
```