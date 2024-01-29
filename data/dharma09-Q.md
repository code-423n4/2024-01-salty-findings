
### [L-01] Wrong condition check violates the rule of liquidity pool

**Details:**
The condition `require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");` checks whether both reserve0 and reserve1 are greater than or equal to the value specified by PoolUtils.DUST. This condition is intended to ensure that the removal of liquidity does not deplete the reserves below a certain threshold, defined by PoolUtils.DUST. The purpose is to prevent the reserves from becoming too low, which could potentially Details the stability and functionality of the liquidity pool.
**Proof of Code:**
- [Pools.sol#L170](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L170C2-L194C1)
```solidity
function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
		{
		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

		(bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);

		// Determine how much liquidity is being withdrawn and round down in favor of the protocol
		PoolReserves storage reserves = _poolReserves[poolID];
		reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
		reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;

		reserves.reserve0 -= uint128(reclaimedA);
		reserves.reserve1 -= uint128(reclaimedB);

		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal"); //@audit require check wrong check reserve0 -> reserve1

		// Switch reclaimed amounts back to the order that was specified in the call arguments so they make sense to the caller
		if (flipped)
			(reclaimedA,reclaimedB) = (reclaimedB,reclaimedA);

		require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

		// Send the reclaimed tokens to the user
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		emit LiquidityRemoved(tokenA, tokenB, reclaimedA, reclaimedB, liquidityToRemove);
		}
```

**Optimized code:**
If you want to correct the condition to ensure that both reserves are above the threshold, you should replace the second part of the condition. The corrected line would be:

```diff
function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
		{
		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );

		(bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);

		// Determine how much liquidity is being withdrawn and round down in favor of the protocol
		PoolReserves storage reserves = _poolReserves[poolID];
		reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
		reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;

		reserves.reserve0 -= uint128(reclaimedA);
		reserves.reserve1 -= uint128(reclaimedB);

		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
-        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity -        removal"); 
+        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity +        removal"); 
		// Switch reclaimed amounts back to the order that was specified in the call arguments so they make sense to the caller
		if (flipped)
			(reclaimedA,reclaimedB) = (reclaimedB,reclaimedA);

		require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );

		// Send the reclaimed tokens to the user
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		emit LiquidityRemoved(tokenA, tokenB, reclaimedA, reclaimedB, liquidityToRemove);
		}
```

### [L-02] potential arithmetic underflow in `_arbitrage()`

**Details:**
Subtraction operation where arbitrageAmountIn is subtracted from amountOut. If amountOut is less than arbitrageAmountIn, it can lead to an underflow, and the result would wrap around to a very large positive number.

Assuming the amountOut is greater than arbitrageAmountIn, the function continues with depositing the arbitrage profit to the DAO and updating stats related to the pools involved in the arbitrage.
These actions, if successfully executed, could indirectly affect the state of the liquidity pool. For example, depositing arbitrage profits to the DAO might Details the available liquidity in the pool or influence the distribution of rewards.

**Proof of Code:**
- [Pools.sol#L283](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L283C2-L298C4)
```solidity
File: src/pools/Pools.sol
284: function _arbitrage(IERC20 arbToken2, IERC20 arbToken3, uint256 arbitrageAmountIn ) internal returns (uint256 arbitrageProfit)
		{ 
		uint256 amountOut = _adjustReservesForSwap( weth, arbToken2, arbitrageAmountIn );
		amountOut = _adjustReservesForSwap( arbToken2, arbToken3, amountOut );
		amountOut = _adjustReservesForSwap( arbToken3, weth, amountOut );

		// Will revert if amountOut < arbitrageAmountIn 
@>		arbitrageProfit = amountOut - arbitrageAmountIn; //@audit

		// Deposit the arbitrage profit for the DAO - later to be divided between the DAO, SALT stakers and liquidity providers in DAO.performUpkeep
 		_userDeposits[address(dao)][weth] += arbitrageProfit;

		// Update the stats related to the pools that contributed to the arbitrage so they can be rewarded proportionally later
		// The arbitrage path can be identified by the middle tokens arbToken2 and arbToken3 (with WETH always on both ends)
		_updateProfitsFromArbitrage( arbToken2, arbToken3, arbitrageProfit );
		}

```

**Optimized code:**
To mitigate this issue, you should perform a check before subtracting to ensure that amountOut is greater than or equal to arbitrageAmountIn. If it's not, you should handle the error appropriately, such as by reverting the transaction.

```diff
File: src/pools/Pools.sol
284: function _arbitrage(IERC20 arbToken2, IERC20 arbToken3, uint256 arbitrageAmountIn ) internal returns (uint256 arbitrageProfit)
		{ 
		uint256 amountOut = _adjustReservesForSwap( weth, arbToken2, arbitrageAmountIn );
		amountOut = _adjustReservesForSwap( arbToken2, arbToken3, amountOut );
		amountOut = _adjustReservesForSwap( arbToken3, weth, amountOut );

+		if (amountout <= arbitrageAmountIn)
+			revert ("InvalidBorrowAmount");
		
		// Will revert if amountOut < arbitrageAmountIn 
+       unchecked {
+        arbitrageProfit = amountOut - arbitrageAmountIn;
+        }
			
		// Deposit the arbitrage profit for the DAO - later to be divided between the DAO, SALT stakers and liquidity providers in DAO.performUpkeep
 		_userDeposits[address(dao)][weth] += arbitrageProfit;

		// Update the stats related to the pools that contributed to the arbitrage so they can be rewarded proportionally later
		// The arbitrage path can be identified by the middle tokens arbToken2 and arbToken3 (with WETH always on both ends)
		_updateProfitsFromArbitrage( arbToken2, arbToken3, arbitrageProfit );
		}
```

### [L-03] Missing checks for ecrecover() signature malleability

**Details:**
ecrecover() accepts as valid, two versions of signatures, meaning an attacker can use the same signature twice, or an attacker may be able to front-run the original signer with the altered version of the signature, causing the signer's transaction to revert due to nonce reuse. Consider adding checks for signature malleability, or using OpenZeppelin's ECDSA library to perform the extra checks necessary in order to prevent malleability.
**Proof of Code:**
- [SigningTools.sol#L13](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L13C5-L28C30)
```solidity
File: src/SiginingTool.sol
11:  function _verifySignature(bytes32 messageHash, bytes memory signature ) internal pure returns (bool)
    	{ 
    	require( signature.length == 65, "Invalid signature length" );

		bytes32 r;
		bytes32 s;
		uint8 v;

		assembly
			{
			r := mload (add (signature, 0x20))
			s := mload (add (signature, 0x40))
			v := mload (add (signature, 0x41))
			}
//@audit-issue Verify s or add a nonce
@>		address recoveredAddress = ecrecover(messageHash, v, r, s);

        return (recoveredAddress == EXPECTED_SIGNER);
    	}
	}
```
### [L-04] Vulnerable versions of packages are being used

**Details:**
The package.json configuration file says that the project is using OZ which has a not last update version. Latest non vulnerable version 5.0.1 for @openzeppelin/contracts

**Proof of Code:**

```solidity
File: lib/chainlink/contracts/package.json
85:    "@openzeppelin/contracts": "~4.3.3",
```

### [L-05] Use block.number instead of block.timestamp
The use of block.timestamp for time-based conditions might be susceptible to miner manipulation. It's generally recommended to use block numbers or other mechanisms that are less susceptible to manipulation.
- [ManagedWallet.sol#L73](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L73C2-L89C4)
```diff
File: src/ManagedWallet.sol
71: // Confirm the wallet proposals - assuming that the active timelock has already expired.
	function changeWallets() external 
		{
		// proposedMainWallet calls the function - to make sure it is a valid address.
		require( msg.sender == proposedMainWallet, "Invalid sender" );
-		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
+		require(block.number >= activeTimelockBlock, "Timelock not yet completed");
		// Set the wallets
		mainWallet = proposedMainWallet;
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet); 
		// Reset
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}
```
### [NC-01] Check to ensure that `proposedMainWallet` is not the zero address before proceeding
the implementation relies on the assumption that the proposedMainWallet has already been validated as a valid address. It would be good practice to include an additional check to ensure that proposedMainWallet is not the zero address before proceeding with the state changes.
- - [ManagedWallet.sol#L73](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L73C2-L89C4)
```
File: src/ManagedWallet.sol
71: // Confirm the wallet proposals - assuming that the active timelock has already expired.
	function changeWallets() external 
		{
		// proposedMainWallet calls the function - to make sure it is a valid address.
		require( msg.sender == proposedMainWallet, "Invalid sender" );
		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

		// Set the wallets
		mainWallet = proposedMainWallet;
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet); 
															
		// Reset
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}
```

