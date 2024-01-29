SALTY GAS OPTIMIZATIONS
--
INTRODUCTION
--
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime.

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include `@audit` tags in comments to facilitate issue explanations."
- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [ Only Make SLOADS when necessary](#g-01-only-make-sloads-when-necessary)
  - [Directly modifies the value in the storage location without reading the current value first](#g-02-directly-modifies-the-value-in-the-storage-location-without-reading-the-current-value-first)
  - [Cache state variable in memory before emiting event](#g-03-cache-state-variable-in-memory-before-emiting-event)
  - [Caching the length of the array can be a good optimization strategy](#g-04-caching-the-length-of-the-array-can-be-a-good-optimization-strategy)
  - [When variables are declared with the storage keyword caching any fields that need to be re-read in stack variables Saves gas](#g-05-when-variables-are-declared-with-the-storage-keyword-caching-any-fields-that-need-to-be-re-read-in-stack-variables-saves-gas)
  - [`ballot` is only used once, no need to cache `proposals.ballotForID(ballotID)`](#g-06-ballot-is-only-used-once-no-need-to-cache-proposalsballotforidballotid)
  - [Use uint8 for Constants and we can save 1 slods by pack with state variable](#g-07-use-uint8-for-constants-and-we-can-save-1-slods-by-pack-with-state-variable)
  - [Move lesser gas costing require checks to the top](#g-08-move-lesser-gas-costing-require-checks-to-the-top)
  - [Add unchecked blocks for subtractions where the operands cannot underflow](#g-09-add-unchecked-blocks-for-subtractions-where-the-operands-cannot-underflow)
  - [Multiple accesses of the same mapping/array key/index should be cached](#g-10-multiple-accesses-of-the-same-mappingarray-keyindex-should-be-cached)
  - [ Refactor the function `changeWallets()` to avoid making unnecessary state loads(SLOADS)](#g-11-refactor-the-function-changewallets-to-avoid-making-unnecessary-state-loadssloads)
  - [Refactor `step11()` for better gas optimization](#g-12-refactor-step11-for-better-gas-optimization)
  - [Cache external calls outside of loop to avoid re-calling function on each iteration](#g-13-cache-external-calls-outside-of-loop-to-avoid-re-calling-function-on-each-iteration)
  - [Enhancing the `_verifySignature` Function in the SigningTools Contract](#g-14-enhancing-the-_verifysignature-function-in-the-signingtools-contract)
  - [ Reduce redundancy and potentially saves gas by avoiding repeated calculations](#g-15-reduce-redundancy-and-potentially-saves-gas-by-avoiding-repeated-calculations)
  - [State variables should be cached in stack variables rather than re-reading them from storage](#g-16-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage)
  - [`numberAuthorized()` is called multiple times -> reduce gas costs by avoiding redundant function calls](#g-17-numberauthorized-is-called-multiple-times---reduce-gas-costs-by-avoiding-redundant-function-calls)
  - [Using storage instead of memory for structs/arrays saves gas](#g-18-using-storage-instead-of-memory-for-structsarrays-saves-gas)

# Gas report

## Table Of Contents

### [G-01] Only Make SLOADS when necessary

**Details:**
In function , `_increseUserShare` the variable data is only used when `require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );`, therefore there's no need to make a state read if we are not going to make use of it

**Proof Of Code**
- [StakingRewards.sol#L56](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L56C2-L86C1)
```solidity
File: src/staking/StakingRewards.sol
57: function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
		{
		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
		require( increaseShareAmount != 0, "Cannot increase zero share" );

		UserShareInfo storage user = _userShareInfo[wallet][poolID]; //@audit cache vaiable when its use write down after below cheack

		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

			// Update the cooldown expiration for future transactions
			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
			}

		uint256 existingTotalShares = totalShares[poolID];

```
If useCooldown is not true than not run second loop, then we save ourselves one entire SLOAD
**Optimized code:**
```diff
57: function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
		{
		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
		require( increaseShareAmount != 0, "Cannot increase zero share" );

-		UserShareInfo storage user = _userShareInfo[wallet][poolID]; 

		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
+       UserShareInfo storage user = _userShareInfo[wallet][poolID]; 				
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

			// Update the cooldown expiration for future transactions
			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
			}

		uint256 existingTotalShares = totalShares[poolID];
```

### [G-02] Directly modifies the value in the storage location without reading the current value first
**Details:**
 using pre-increment `(++i)` instead of the addition assignment `(i += 1)` can sometimes result in more efficient code. By doing this we can avoid state variable read as a result we can save gas.
**Proof of Code:**
- [AccessManager.sol#L40](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L40C1-L46C7)
```solidity
File: src/AccessManager.sol
41:     function excludedCountriesUpdated() external
    	{
    	require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

        geoVersion += 1; //@audit Use pre Increment
    	}

```
The idea here is that using `++i` directly modifies the value in the storage location without reading the current value first, potentially saving gas compared to `i += 1`, which involves reading the current value, adding 1, and then writing it back.

**Optimized code:**
```diff
41:     function excludedCountriesUpdated() external
    	{
    	require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );

-        geoVersion += 1; 
+	     geoVersion ++;
    	}
```

### [G-03] Cache state variable in memory before emiting event

**Details:**
geoVersion is assigned the value of geoVersion before emitting the AccessGranted event. This can potentially save gas if the compiler doesn't optimize away the redundant read operation.
**Proof of Code:**
- [AccessManager.sol#L67](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L67C6-L67C54)
```solidity
File: src/AccessManager.sol
61:  function grantAccess(bytes calldata signature) external
    	{
    	require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );

        _walletsWithAccess[geoVersion][msg.sender] = true;

        emit AccessGranted( msg.sender, geoVersion ); //@audit do need emit state variable
    	}
```

**Optimized code:**
assigns the value of `_geoVersion` to the geoVersion state variable.
```diff
File: src/AccessManager.sol
61:  function grantAccess(bytes calldata signature) external
    	{
    	require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );

        _walletsWithAccess[geoVersion][msg.sender] = true;
+       geoVersion = _geoVersion;

-       emit AccessGranted( msg.sender, geoVersion ); 
+       emit AccessGranted( msg.sender, _geoVersion ); 
    	}
```
In function `incrementBurnableUSDS` use two time state variable which can avoid by caching in stack variable.
**Proof of Code:**
- [Liquidizer.sol#L134](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L134C3-L149C5)
```diff
File: src/stable/Liquidizer.sol
81: 	function incrementBurnableUSDS( uint256 usdsToBurn ) external
		{
		require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
+       _usdsThatShouldBeBurned = usdsThatShouldBeBurned;
-		usdsThatShouldBeBurned += usdsToBurn; //@audit usdsThatShouldBeBurned
+       usdsThatShouldBeBurned = _usdsThatShouldBeBurned + usdsToBurn; 

-		emit incrementedBurnableUSDS(usdsThatShouldBeBurned);
+		emit incrementedBurnableUSDS(_usdsThatShouldBeBurned);
		}
```

### [G-04] Caching the length of the array can be a good optimization strategy

**Details:**
By caching the length, you save gas by eliminating the need to repeatedly access the length property of the array.This is particularly beneficial if the length of the array doesn't change during the loop execution.
**Proof of Code:**
cache `userUnstakes.length;` in `unstakesForUser`
- [Staking.sol#L150](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L150C2-L167C6)
```diff
File: src/staking/Staking.sol
150: function unstakesForUser( address user, uint256 start, uint256 end ) public view returns (Unstake[] memory)
		{
        // Check if start and end are within the bounds of the array
        require(end >= start, "Invalid range: end cannot be less than start");

        uint256[] memory userUnstakes = _userUnstakeIDs[user]; 
+       uint256 userUnstakesLength = userUnstakes.length;
-        require(userUnstakes.length > end, "Invalid range: end is out of bounds"); //@audit cache userUnstake.length
-        require(start < userUnstakes.length, "Invalid range: start is out of bounds");
+        require(userUnstakesLength > end, "Invalid range: end is out of bounds"); 
+        require(start < userUnstakesLength, "Invalid range: start is out of bounds");

        Unstake[] memory unstakes = new Unstake[](end - start + 1);

        uint256 index; 
        for(uint256 i = start; i <= end; i++) 
            unstakes[index++] = _unstakesByID[ userUnstakes[i]];

        return unstakes;
    }
```
Cache `unstakeIDs.length` in `unstakesForUser`
**Proof Of Code**
- [Staking.sol#L171](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L171C2-L179C4)
```diff
File: src/staking/Staking.sol
171: 	function unstakesForUser( address user ) external view returns (Unstake[] memory)
		{
		// Check to see how many unstakes the user has
		uint256[] memory unstakeIDs = _userUnstakeIDs[user]; 
-		if ( unstakeIDs.length == 0 )
+		if ( unstakeIDs == 0 )
			return new Unstake[](0);

		// Return them all
-		return unstakesForUser( user, 0, unstakeIDs.length - 1 ); //@audit cache length
+		return unstakesForUser( user, 0, unstakeIDs - 1 );
		}
```
### [G-05] When variables are declared with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas

**Details:**
Use store values in local variables before emitting the event.This way we can avoid to read state variable two time.
**Proof of Code:**
- [ManagedWallet.sol#L42](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L42C2-L55C4)
```solidity
File: src/ManagedWallet.sol
42: function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
		{
		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );

		// Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." ); 

		proposedMainWallet = _proposedMainWallet;
		proposedConfirmationWallet = _proposedConfirmationWallet;

		emit WalletProposal(proposedMainWallet, proposedConfirmationWallet); //@audit use memory value
		}
```

**Optimized code:**
By doing this, you avoid reading the state variables twice when emitting the event, potentially saving some gas. 
```diff
42: function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
		{
		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );

		// Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." ); 

		proposedMainWallet = _proposedMainWallet;
		proposedConfirmationWallet = _proposedConfirmationWallet;

-		emit WalletProposal(proposedMainWallet, proposedConfirmationWallet); 
+		emit WalletProposal(_proposedMainWallet, _proposedConfirmationWallet);
		}
```
### [G-06] `ballot` is only used once, no need to cache `proposals.ballotForID(ballotID)`
**Details:**
It makes no sense to cache data in memory if a state variable is only used once throughout the entire function. We can save one sold by using the state variable directly.
**Proof of Code:**
- [DAO.sol#L219](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L219C2-L230C1)
```solidity
File: src/dao/Dao.sol
219: function _finalizeApprovalBallot( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
			Ballot memory ballot = proposals.ballotForID(ballotID); //@audit do not cache state variable if used once
			_executeApproval( ballot );
			}

		proposals.markBallotAsFinalized(ballotID);
		}
```
**Optimized code:**
We can use `proposals.ballotForID(ballotID` Directly there's no need to cache.
```diff
219: function _finalizeApprovalBallot( uint256 ballotID ) internal
		{
		if ( proposals.ballotIsApproved(ballotID ) )
			{
-			Ballot memory ballot = proposals.ballotForID(ballotID);
-			_executeApproval( ballot );
+			_executeApproval( proposals.ballotForID(ballotID) );
			}

		proposals.markBallotAsFinalized(ballotID);
		}
```
### [G-07] Use uint8 for Constants and we can save 1 slods by pack with state variable
Change uint256 to uint8 for PERCENT_POL_TO_WITHDRAW if the value fits within the range of a uint8.

**Proof Of Code**
- [Liquidizer.sol#L29](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L29)
```diff
File: src/stable/Liquidizer.sol
-29: uint256 constant PERCENT_POL_TO_WITHDRAW = 1;
+29: uint8 constant PERCENT_POL_TO_WITHDRAW = 1;
```
```
uint256 //1 
	uint8 constant PERCENT_POL_TO_WITHDRAW = 1; //1 1
    IERC20 immutable public wbtc; //20 1
    IERC20 immutable public weth; //20 2
    IUSDS immutable public usds;  //32 3
    ISalt immutable public salt; //32 4
    IERC20 immutable public dai; //20 5

```

### [G-08] Move lesser gas costing require checks to the top

**Refactor `withdraw` function to save more gas**	
Reorder the checks to have cheaper checks first

**Details:**
Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
**Proof of Code:**
- [Pools.sol#L219](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L219C4-L230C7)
```solidity
File: src/pools/Pools.sol
220:   function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small"); //@audit check before above conditoin its cheaper to above check

		_userDeposits[msg.sender][token] -= amount; 

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

**Optimized code:**

```diff
220:   function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
-   	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");
+   	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );

		_userDeposits[msg.sender][token] -= amount; 

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```
**Details**
In `Pools.sol::removeLiquidity` check first  `msg.sender == address(collateralAndLiquidity)` and this check involve with global value which consume more gas then other check. so its recommended to revert or check first cheaper statement. This way we can save some gas by reverting cheaper condition.
- [Pools.sol#L170](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L170C2-L201C1)
**Optimized code**
```diff
File:src/pools/Pools.sol
170:    function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
		{ 
-		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" ); //@audit check input value first
+		require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
		(bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);

		// Determine how much liquidity is being withdrawn and round down in favor of the protocol
		PoolReserves storage reserves = _poolReserves[poolID];
		reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
		reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;

		reserves.reserve0 -= uint128(reclaimedA);
		reserves.reserve1 -= uint128(reclaimedB);

		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
		// one of token has more decimal than other
		// Make sure that the reserves after swap are above DUST
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
**Details**
In `CollateralAndLiquidity.sol::repayUSDS` check first  ` userShareForPool( msg.sender, collateralPoolID` and `amountRepaid <= usdsBorrowedByUsers[msg.sender]` which envolve state variable check which consume more gas than input value check. so its recommended to revert or check first cheaper statement.check `amountRapid > 0` This way we can save some gas by reverting cheaper condition.
- [CollateralAndLiquidity.sol#L115](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115C5-L135C4)
**proof of Code**

```diff
File: src/stable/CollateralAndLiquidity.sol
115:  function repayUSDS( uint256 amountRepaid ) external nonReentrant
		{
-		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
-		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		require( amountRepaid > 0, "Cannot repay zero amount" ); //@audit check first
+		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
+		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		// Decrease the borrowed amount for the user
		usdsBorrowedByUsers[msg.sender] -= amountRepaid; 

		// Have the user send the USDS to the USDS contract so that it can later be burned (on USDS.performUpkeep)
		usds.safeTransferFrom(msg.sender, address(usds), amountRepaid);

		// Have USDS remember that the USDS should be burned
		liquidizer.incrementBurnableUSDS( amountRepaid );

		// Check if the user no longer has any borrowed USDS
		if ( usdsBorrowedByUsers[msg.sender] == 0 ) 
			_walletsWithBorrowedUSDS.remove(msg.sender);

		emit RepaidUSDS(msg.sender, amountRepaid);
		}
```
**Details**
In `USDS.sol::mintTo` involve some pre check in this condition function check first require statement which chekck msg.sender against state variable which is consume more gas. So its better to check after input value check.
- [USDS.sol#L40](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L40C2-L50C1)
**Proof of Code**
```diff
File: src/stable/USDS.sol
40: function mintTo( address wallet, uint256 amount ) external
		{
-		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
		require( amount > 0, "Cannot mint zero USDS" ); //@audit check first
+		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
		_mint( wallet, amount );

		emit USDSMinted(wallet, amount);
		}
```
**Details**
in `StakingRewards.sol::_increaseShareAmount` function check pool is whiltelisted or not which contain external call and this require statment cost more gas. So its better to revert after input value check.Which save huge gas by avoiding  external call first.
- [StakingRewards.sol#L57](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L57C2-L80C19)
**Proof Of Code**
```diff
File: src/staking/StakingRewards.sol
57: function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
		{ 
-		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
		require( increaseShareAmount != 0, "Cannot increase zero share" ); //@audit check first
+		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
		UserShareInfo storage user = _userShareInfo[wallet][poolID];

		if ( useCooldown )
		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

			// Update the cooldown expiration for future transactions
			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
			}

		uint256 existingTotalShares = totalShares[poolID];

```
### [G-09] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.

### Please note this instance was not included in the bots reports
**Details:**
In `Pools.sol::withdraw` function update balance after user withdraw which happen on l225 and also doing check for  `_userDeposits[msg.sender][token] >= amount` on line 222. That means we can wrap this statement in unchecked block for saving some gas.
**Proof of Code:**
- [Pools.sol#L220](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L219C5-L231C1)
```solidity
File: src/pools/Pools.sol
220:  function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount; //@audit unchecked this block

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

**Optimized code:**

```diff
220:   function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small"); 
+		unchecked {
+		_userDeposits[msg.sender][token] -= amount;
+        }
    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```	
**Details**
In `StakingRewards.sol` function update claimableRewards after user claim which happen on l134 and also doing check for  `virtualRewardsToRemove < rewardsForAmount ` on line 133. That means we can wrap this statement in unchecked block for saving some gas.
- [StakingRewards.sol#L130](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L129C1-L141C1)
**Proof Of Code**
```diff
File: src/staking/StakingRewards.sol
130: 	// Some of the rewardsForAmount are actually virtualRewards and can't be claimed.
		// In the event that virtualRewards are greater than actual rewards - claimableRewards will stay zero.
		if ( virtualRewardsToRemove < rewardsForAmount )
			claimableRewards = rewardsForAmount - virtualRewardsToRemove; //@audit unchecked 

		// Send the claimable rewards
```
**Details**
In `CollateralAndLiquidity.sol::repayUSDS` function update borrow share after user borrow amount which happen on L120 and also doing check for  `amountRepaid <= usdsBorrowedByUsers[msg.sender]` on line 118. That means we can wrap this statement in unchecked block for saving some gas.
**proof of code**
- [CollateralAndLiquidity.sol#L115](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115C5-L135C4)
```diff
File: src/stable/CollateralAndLiquidity.sol
115:  function repayUSDS( uint256 amountRepaid ) external nonReentrant
		{
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
		require( amountRepaid > 0, "Cannot repay zero amount" );

		// Decrease the borrowed amount for the user
-		usdsBorrowedByUsers[msg.sender] -= amountRepaid; //@audit unchecked this block
+		unchecked{
+		usdsBorrowedByUsers[msg.sender] -= amountRepaid;
+		}

		// Have the user send the USDS to the USDS contract so that it can later be burned (on USDS.performUpkeep)
		usds.safeTransferFrom(msg.sender, address(usds), amountRepaid);

		// Have USDS remember that the USDS should be burned
		liquidizer.incrementBurnableUSDS( amountRepaid );

		// Check if the user no longer has any borrowed USDS
		if ( usdsBorrowedByUsers[msg.sender] == 0 )
			_walletsWithBorrowedUSDS.remove(msg.sender);

		emit RepaidUSDS(msg.sender, amountRepaid);
		}
```
**Details**
In `Pools.sol::swap` function update balance after user swap which happen on L378 and also doing check for  `userDeposits[swapTokenIn] >= swapAmountIn` on line 377. That means we can wrap this statment in unchecked block for saving some gas.
- [Pools.sol#L372](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L371C6-L379C1)
**Optimized code:**
```diff
File: src/pools/Pools.sol
372: function swap( IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
		{ 
		// Confirm and adjust user deposits
		mapping(IERC20=>uint256) storage userDeposits = _userDeposits[msg.sender];

    	require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" );
-		userDeposits[swapTokenIn] -= swapAmountIn; 
+		unchecked {
+ 		userDeposits[swapTokenIn] -= swapAmountIn;
+		}
		swapAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenOut, swapAmountIn, minAmountOut );

		// Deposit the final tokenOut for the caller
		userDeposits[swapTokenOut] += swapAmountOut;
		}
```
### [G-10] Multiple accesses of the same mapping/array key/index should be cached
**note : All instances are not found by bot race**

**Details:**
Repeatedly accessing the same mapping or array key/index in smart contracts without caching can result in substantial gas consumption. In Solidity, each read operation from storage incurs a gas cost. By implementing a caching mechanism for recurring accesses, significant gas savings can be achieved. This is particularly crucial in loops or functions with multiple reads of the same data.

**Proof of Code:**
- [Pools.sol#L219](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L219C4-L230C7)
```solidity
File: src/pools/Pools.sol
220:  function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small"); 

		_userDeposits[msg.sender][token] -= amount; //@audit Cache _userDeposits[msg.sender][token] 

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

**Optimized code:**

```diff
220:    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
+      UserAmount calldata _userAmount = _userDeposits[msg.sender][token];

-    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
+    	require( _userDeposits >= amount, "Insufficient balance to withdraw specified amount" );
        require( amount > PoolUtils.DUST, "Withdraw amount too small"); 

-		_userDeposits[msg.sender][token] -= amount; 
+		_userDeposits[msg.sender][token] = _userAmount - amount;
    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```
**Proof Of Code**
cache `totalShares[poolID]` in `StakingRewards.sol::userRewardForPool()` 
- [StakingRewards.sol#L232](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L232C2-L254C1)
```diff
File: src/staking/StakingRewards.sol
232: function userRewardForPool( address wallet, bytes32 poolID ) public view returns (uint256)
		{ 
+       TotalShare memory totalShares = totalShares[poolID];
		// If there are no shares for the pool, the user can't have any shares either and there can't be any rewards
-		if ( totalShares[poolID] == 0 )
+		if ( totalShares == 0 )
			return 0;

		UserShareInfo memory user = _userShareInfo[wallet][poolID]; 
		if ( user.userShare == 0 )
			return 0;

		// Determine the share of the rewards for the user based on their deposited share
-		uint256 rewardsShare = ( totalRewards[poolID] * user.userShare ) / totalShares[poolID]; //@audit cache totalShares[poolID]
+		uint256 rewardsShare = ( totalRewards[poolID] * user.userShare ) / totalShares; 
		// Reduce by the virtualRewards - as they were only added to keep the share / rewards ratio the same when the used added their share

		// In the event that virtualRewards exceeds rewardsShare due to precision loss - just return zero
		if ( user.virtualRewards > rewardsShare )
			return 0;

		return rewardsShare - user.virtualRewards;
		}
```
**Proof Of Code**
Cache `totalShares[poolID]` abd `totoalRewards[poolID]` in 	`StakingRewards.sol`
- [StakingRewards.sol#L104](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L104C3-L116C23)
```diff
File: src/staking/StakingRewards.sol
105: if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
			{
			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

			// Update the cooldown expiration for future transactions
			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
			}
+		TotalRewards memory totalRewards = totalRewards[poolID];
+		TotalShares memory totalShares = totalShares[poolID];
		// Determine the share of the rewards for the amountToDecrease (will include previously added virtual rewards)
-		uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID]; //@audit cache totoalRewards[poolID] and totalShares[poolID]
+		uint256 rewardsForAmount = ( totalRewards * decreaseShareAmount ) / totalShares;

		// For the amountToDecrease determine the proportion of virtualRewards (proportional to all virtualRewards for the user)
		// Round virtualRewards down in favor of the protocol
		uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;

		// Update totals
-		totalRewards[poolID] -= rewardsForAmount;
-		totalShares[poolID] -= decreaseShareAmount;
+		totalRewards[poolID] = totalRewards - rewardsForAmount;
+		totalShares[poolID] = totalShares - decreaseShareAmount;
		// Update the user's share and virtual rewards
		user.userShare -= uint128(decreaseShareAmount);
		user.virtualRewards -= uint128(virtualRewardsToRemove);

		uint256 claimableRewards = 0; 


```
Cache `totalRewards[poolID]` in `StakingRewards.sol::`. also in this function there's calculation for virtualRewardsToAdd which multiplcation with totalRewars[poolID]. If we use memory value than we can save some gas.
- [StakingRewards.sol#L64](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L64C3-L80C18)
**Proof OF Code**
```diff
File: src/staking/StakingRewards.sol
73: 	uint256 existingTotalShares = totalShares[poolID];

		// Determine the amount of virtualRewards to add based on the current ratio of rewards/shares.
		// The ratio of virtualRewards/increaseShareAmount is the same as totalRewards/totalShares for the pool.
		// The virtual rewards will be deducted later when calculating the user's owed rewards.
        if ( existingTotalShares != 0 ) // prevent / 0
        	{
			// Round up in favor of the protocol.
-			uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards[poolID] * increaseShareAmount, existingTotalShares ); //@audit Cache totalRewards[poolID]
+			uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards * increaseShareAmount, existingTotalShares ); 

			user.virtualRewards += uint128(virtualRewardsToAdd);
-	        totalRewards[poolID] += uint128(virtualRewardsToAdd);
+	        totalRewards[poolID] = totalRewards + uint128(virtualRewardsToAdd);
	        }

		// Update the deposit balances
		user.userShare += uint128(increaseShareAmount);
		totalShares[poolID] = existingTotalShares + increaseShareAmount;

		emit UserShareIncreased(wallet, poolID, increaseShareAmount);
		}
```
First of all cache `pendingRewards[poolID]` and make variable outside for loop. with help of this we can skip the variable creation in every interaction.
- [RewardsEmitter.sol#L111](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L111C3-L134C1)
**Proof Of code**
```diff
File: src/rewards/RewardsEmitter.sol
112: // Rewards to emit = pendingRewards * timeSinceLastUpkeep * rewardsEmitterDailyPercent / oneDay
		uint256 numeratorMult = timeSinceLastUpkeep * rewardsConfig.rewardsEmitterDailyPercentTimes1000();
		uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000

+       PendingRewards memory pendingRewards = pendingRewards[poolID];		
+       bytes32 poolID;
		uint256 sum = 0;
		for( uint256 i = 0; i < poolIDs.length; i++ )
			{
-			 poolID = poolIDs[i]; //@audit make variable outside

			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
-			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult; //@audit cache pendingRewards[poolID] 
+           uint256 amountToAddForPool = ( pendingRewards* numeratorMult ) / denominatorMult;
			// Reduce the pending rewards so they are not sent again
			if ( amountToAddForPool != 0 )
				{
-				pendingRewards[poolID] -= amountToAddForPool;
+				pendingRewards[poolID] = pendingRewards[poolID] - amountToAddForPool;

				sum += amountToAddForPool;
				}

			// Specify the rewards that will be added for the specific pool
			addedRewards[i] = AddedReward( poolID, amountToAddForPool );
			}

		// Add the rewards so that they can later be claimed by the users proportional to their share of the StakingRewards derived contract.
		stakingRewards.addSALTRewards( addedRewards );
		}

```
Cache `usdsBorrowedByUsers[msg.sender]`
- [CollateralAndLiquidity.sol#L115](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115C5-L135C4)
**Proof Of Code**
```diff
File: src/stable/CollateralAndLiquidity.sol
115: function repayUSDS( uint256 amountRepaid ) external nonReentrant
		{
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
-        UsdsBorrowedByUsers memory  usdsBorrowedByUsers = usdsBorrowedByUsers[msg.sender];
-		require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" ); //@audit cache usdsBorrowedByUsers[msg.sender]
+		require( amountRepaid <= usdsBorrowedByUsers, "Cannot repay more than the borrowed amount" ); 
		require( amountRepaid > 0, "Cannot repay zero amount" ); 

		// Decrease the borrowed amount for the user
-		usdsBorrowedByUsers[msg.sender] -= amountRepaid;
+		usdsBorrowedByUsers[msg.sender] = usdsBorrowedByUsers - amountRepaid;

		// Have the user send the USDS to the USDS contract so that it can later be burned (on USDS.performUpkeep)
		usds.safeTransferFrom(msg.sender, address(usds), amountRepaid);

		// Have USDS remember that the USDS should be burned
		liquidizer.incrementBurnableUSDS( amountRepaid );

		// Check if the user no longer has any borrowed USDS
		if ( usdsBorrowedByUsers[msg.sender] == 0 ) 
			_walletsWithBorrowedUSDS.remove(msg.sender);

		emit RepaidUSDS(msg.sender, amountRepaid);
		}
```

### [G-11] Refactor the function `changeWallets()` to avoid making unnecessary state loads(SLOADS)
**Details:**
Change wallet functionality, which updates the state variable, is implemented in the `changeWallets()` function of `ManagedWallet.sol`. In order to prevent needless state changes, we can first make sure the input value hasn't changed before reverting.
**Proof of Code:**
- [ManagedWallet.sol#L73](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L73C2-L89C4)
```solidity
File: src/ManagedWallet.sol
73:	function changeWallets() external
		{
		// proposedMainWallet calls the function - to make sure it is a valid address.
		require( msg.sender == proposedMainWallet, "Invalid sender" );
		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

		// Set the wallets
		mainWallet = proposedMainWallet; //@audit 
		confirmationWallet = proposedConfirmationWallet;

		emit WalletChange(mainWallet, confirmationWallet); 
															
		// Reset
		activeTimelock = type(uint256).max;
		proposedMainWallet = address(0);
		proposedConfirmationWallet = address(0);
		}
```

**Optimized code:**

```diff
73: function changeWallets() external {
    require(msg.sender == proposedMainWallet, "Invalid sender");
    require(block.timestamp >= activeTimelock, "Timelock not yet completed");

-		// Set the wallets
-		mainWallet = proposedMainWallet;
-		confirmationWallet = proposedConfirmationWallet;

-       emit WalletChange(mainWallet, confirmationWallet);
+    // Check if wallets are different before changing state
+    if (mainWallet != proposedMainWallet || confirmationWallet != proposedConfirmationWallet) {
+        // Set the wallets
+        mainWallet = proposedMainWallet;
+        confirmationWallet = proposedConfirmationWallet;

+        emit WalletChange(mainWallet, confirmationWallet);
+   }

    // Reset
    activeTimelock = type(uint256).max;
    proposedMainWallet = address(0);
    proposedConfirmationWallet = address(0);
}

```
### [G-12] Refactor `step11()` for better gas optimization

**Details:**
Combine the two external calls to the VestingWallet contract into a single call. This can save gas by reducing the number of external calls.

By doing this, we save gas by avoiding the repetition of the external call to the same contract.
**Proof of Code:**
- [Upkeep.sol#L231](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L231C2-L240C1)
```solidity
File: src/Upkeep.sol
231: function step11() public onlySameContract
		{
		uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt)); //@audit two external call

		// teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
		VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));

		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
		}

```
**Optimized code:**
we can cache external call in local variable
```diff
function step11() public onlySameContract
		{
-		uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt)); 
+        VestingWallet teamVestingWallet = VestingWallet(payable(exchangeConfig.teamVestingWallet()));
+			uint256 releaseableAmount = teamVestingWallet.releasable(address(salt));
		
		// teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
-		VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
+			teamVestingWallet.release(address(salt));
		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
		}

```
### [G-13] Cache external calls outside of loop to avoid re-calling function on each iteration

**Details:**
A loop is implemented in the `findLiquidatableUsers` function to determine which user has at least mincollateral. In the meantime, the calculation calls an external function on `line 26`, which calls each interaction and wastes gas.For better gas efficiency Cache Function Calls outside for loop.
**Proof of Code:**
- [CollateralAndLiquidity.sol#L310](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L310C1-L334C6)
```solidity
File: src/stable/CollateralAndLiquidity.sol
311: function findLiquidatableUsers( uint256 startIndex, uint256 endIndex ) public view returns (address[] memory)
		{
		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);
		uint256 count = 0;

		// Cache
		uint256 totalCollateralShares = totalShares[collateralPoolID];
		uint256 totalCollateralValue = totalCollateralValueInUSD();

		if ( totalCollateralValue != 0 )
			for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
				address wallet = _walletsWithBorrowedUSDS.at(i);

				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100; //@audit cache external call

				// Determine minCollateral in terms of minCollateralValue
				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;

				// Make sure the user has at least minCollateral
				if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )
					liquidatableUsers[count++] = wallet;
				}

		// Resize the array to match the actual number of liquidatable positions found
		address[] memory resizedLiquidatableUsers = new address[](count);
		for ( uint256 i = 0; i < count; i++ )
			resizedLiquidatableUsers[i] = liquidatableUsers[i];

		return resizedLiquidatableUsers;
		}
```
**Optimized code:**
```diff
311: function findLiquidatableUsers( uint256 startIndex, uint256 endIndex ) public view returns (address[] memory)
		{
		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);
		uint256 count = 0;

		// Cache
		uint256 totalCollateralShares = totalShares[collateralPoolID];
		uint256 totalCollateralValue = totalCollateralValueInUSD();
+       minimumCOllateralRatioPercent = stableConfig.minimumCollateralRatioPercent();
		if ( totalCollateralValue != 0 )
			for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
				address wallet = _walletsWithBorrowedUSDS.at(i);

				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * minimumCollateralRatioPercent) / 100;
				// Determine minCollateral in terms of minCollateralValue
				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;

				// Make sure the user has at least minCollateral
				if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )
					liquidatableUsers[count++] = wallet;
				}

		// Resize the array to match the actual number of liquidatable positions found
		address[] memory resizedLiquidatableUsers = new address[](count);
		for ( uint256 i = 0; i < count; i++ )
			resizedLiquidatableUsers[i] = liquidatableUsers[i];

		return resizedLiquidatableUsers;
		}
```
The `poolsConfig.isWhitelisted` function is an external call within the loop. External calls can be expensive in terms of gas. If possible, consider refactoring the logic to perform the external call outside the loop and store the result in a local variable. This way, you can avoid making the same external call multiple times.
- [RewardsEmitter.sol#L57](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L57C2-L72C5)
**Proof of code**
```solidity
File: src/rewards/RewardsEmitter.sol
57: function addSALTRewards( AddedReward[] calldata addedRewards ) external nonReentrant
		{
		uint256 sum = 0;
		for( uint256 i = 0; i < addedRewards.length; i++ )
			{
			AddedReward memory addedReward = addedRewards[i];
@>			require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" ); //@audit external call in loop

			uint256 amountToAdd = addedReward.amountToAdd;
			if ( amountToAdd != 0 )
				{
				// Update pendingRewards so the SALT can be distributed later
				pendingRewards[ addedReward.poolID ] += amountToAdd;
				sum += amountToAdd;
				}
			}

		// Transfer the SALT from the sender for all of the specified rewards
		if ( sum > 0 )
			salt.safeTransferFrom( msg.sender, address(this), sum );
		}
```
### [G-14]  Enhancing the `_verifySignature` Function in the SigningTools Contract

**Details:**
The _verifySignature function in the SigningTools contract uses inline assembly to split an Ethereum signature into its components (r, s, v).On line20 function Extracting r, s, and v from the signature using assembly. While inline assembly can be efficient in terms of gas usage, it's generally more error-prone and can introduce security vulnerabilities if not implemented correctly. This report suggests a refactoring of the _verifySignature function to use Solidity's native operations, potentially improving security and maintainability with minimal Details on gas costs.

The optimized code removes the use of inline assembly for extracting the v component of the signature. Instead, it directly accesses the 65th byte of the signature array. This approach leverages Solidity's native array handling, which is generally safer and less prone to errors compared to manual memory manipulation in assembly.
The v value is adjusted to comply with Ethereum's signature malleability rules (EIP-155), ensuring it falls within the correct range.

**Proof of Code:**
- [SigningTools.sol#L11](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L11C5-L29C7)
```solidity
File: src/SigningTools.sol
11:    function _verifySignature(bytes32 messageHash, bytes memory signature ) internal pure returns (bool)
    	{ 
    	require( signature.length == 65, "Invalid signature length" );

		bytes32 r;
		bytes32 s;
		uint8 v;

		assembly
			{
			r := mload (add (signature, 0x20))
			s := mload (add (signature, 0x40))
			v := mload (add (signature, 0x41)) //@audit
			}

		address recoveredAddress = ecrecover(messageHash, v, r, s);

        return (recoveredAddress == EXPECTED_SIGNER);
    	}
```

**Optimized code:**

```diff
11:    function _verifySignature(bytes32 messageHash, bytes memory signature ) internal pure returns (bool)
    	{ 
    	require( signature.length == 65, "Invalid signature length" );

		bytes32 r;
		bytes32 s;
		uint8 v;

		assembly
			{
			r := mload (add (signature, 0x20))
			s := mload (add (signature, 0x40))
-			v := mload (add (signature, 0x41))
			}
+      //directly accesses the 65th byte of the signature array
+    	v = uint8(signature[64]);
+	    // Adjust v for Ethereum's malleability rules
+	    if (v < 27) v += 27;

		address recoveredAddress = ecrecover(messageHash, v, r, s);

        return (recoveredAddress == EXPECTED_SIGNER);
    	}
```

### [G-15] Reduce redundancy and potentially saves gas by avoiding repeated calculations

**Details:**
We can further optimize the code by calculating the common part only once and reusing it.

the results of the external calls (totalStaked, baseQuorumTimes1000, and totalSupply) are cached in local variables, reducing the number of external calls made. Caching external calls can significantly optimize gas usage, especially when they involve complex calculations or storage reads.
**Proof of Code:**
- [Proposals.sol#L317](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L317C2-L339C4)
```solidity
File: src/dao/Proposals.sol
317:	function requiredQuorumForBallotType( BallotType ballotType ) public view returns (uint256 requiredQuorum)
		{
		// The quorum will be specified as a percentage of the total amount of SALT staked
		uint256 totalStaked = staking.totalShares( PoolUtils.STAKED_SALT );
		require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );

		if ( ballotType == BallotType.PARAMETER )
			requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 ); //@audit cache this calculation
		else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
			requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
		else
			// All other ballot types require 3x multiple of the baseQuorum
			requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

		// Make sure that the requiredQuorum is at least 0.50% of the total SALT supply.
		// Circulating supply after the first 45 days of emissions will be about 3 million - so this would require about 16% of the circulating
		// SALT to be staked and voting to pass a proposal (including whitelisting) 45 days after deployment..
		uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
		uint256 minimumQuorum = totalSupply * 5 / 1000;

		if ( requiredQuorum < minimumQuorum )
			requiredQuorum = minimumQuorum;
		}
```

**Optimized code:**
the calculation is cached in the variable quorum, which is then reused in each branch of the conditional statements. 
```diff
function requiredQuorumForBallotType(BallotType ballotType) public view returns (uint256 requiredQuorum) {
    
    // The quorum will be specified as a percentage of the total amount of SALT staked
    uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
    require(totalStaked != 0, "SALT staked cannot be zero to determine quorum");

+   uint256 quorum = totalStaked * daoConfig.baseBallotQuorumPercentTimes1000() / (100*1000);
-	if ( ballotType == BallotType.PARAMETER )
-			requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 ); 
-		else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
-			requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
-		else
-			// All other ballot types require 3x multiple of the baseQuorum
-			requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

+  if (ballotType == BallotType.PARAMETER) {
+        requiredQuorum = (1 *  quorum);
+    } else if (ballotType == BallotType.WHITELIST_TOKEN || ballotType == BallotType.UNWHITELIST_TOKEN) {
+        requiredQuorum = (2 * quorum);
+    } else {
+        // All other ballot types require 3x multiple of the baseQuorum
+        requiredQuorum = (3 * quorum);
+    }

    // Cache external call result
    ERC20 saltToken = ERC20(address(exchangeConfig.salt()));
    uint256 totalSupply = saltToken.totalSupply();

    // Cache external call result
    uint256 minimumQuorum = totalSupply * 5 / 1000;

    // Use cached value
    if (requiredQuorum < minimumQuorum)
        requiredQuorum = minimumQuorum;
}

```

### [G-16] State variables should be cached in stack variables rather than re-reading them from storage

**Details:**
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

**Please note these instances were not included in the bots reports**
**Proof of Code:**
- [Airdrop.sol#L74](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L74C5-L86C1)
```solidity
File: src/launch/Airdrop.sol
74:   function claimAirdrop() external nonReentrant
    	{
    	require( claimingAllowed, "Claiming is not allowed yet" );
    	require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
    	require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );

		// Have the Airdrop contract stake a specified amount of SALT and then transfer it to the user
		staking.stakeSALT( saltAmountForEachUser );
		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser ); //@audit saltAmountEachUser

    	claimed[msg.sender] = true;
    	}
```

**Proof of code:**

```diff
File: src/stable/Liquidizer.sol
101: function _possiblyBurnUSDS() internal
		{
+		_usdsThatShouldBeBurned = usdsThatShouldBeBurned;

		// Check if there is USDS to burn
-		if ( usdsThatShouldBeBurned == 0 ) 
+		if ( _usdsThatShouldBeBurned == 0 ) 
			return;

		uint256 usdsBalance = usds.balanceOf(address(this));
-		if ( usdsBalance >= usdsThatShouldBeBurned )
+		if ( usdsBalance >= _usdsThatShouldBeBurned )
			{
			// Burn only up to usdsThatShouldBeBurned.
			// Leftover USDS will be kept in this contract in case it needs to be burned later.
-			_burnUSDS( usdsThatShouldBeBurned );
+			_burnUSDS( _usdsThatShouldBeBurned );
    		usdsThatShouldBeBurned = 0;
			}
		else
			{
			// The entire usdsBalance will be burned - but there will still be an outstanding balance to burn later
			_burnUSDS( usdsBalance );
-			usdsThatShouldBeBurned -= usdsBalance;
+			usdsThatShouldBeBurned = _usdsThatShouldBeBurned - usdsBalance;

			// As there is a shortfall in the amount of USDS that can be burned, liquidate some Protocol Owned Liquidity and
			// send the underlying tokens here to be swapped to USDS
			dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
			dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);
			}
		}
```
- [PriceAggregator.sol#L64](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L64C1-L79C4)
**Poof Of Code**
```diff
File: src/price_feed/PriceAggregator.sol
65: function changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase) public onlyOwner
		{
+			_maximumPriceFeedPercentDifferenceTimes1000 = maximumPriceFeedPercentDifferenceTimes1000;
        if (increase)
            {
-            if (_maximumPriceFeedPercentDifferenceTimes1000 < 7000) //@audit cache maximumPriceFeedPercentDifferenceTimes1000
+            if (_maximumPriceFeedPercentDifferenceTimes1000 < 7000)
-                maximumPriceFeedPercentDifferenceTimes1000 += 500;
+                maximumPriceFeedPercentDifferenceTimes1000 = maximumPriceFeedPercentDifferenceTimes1000 + 500;
            }
        else
            {
-            if (maximumPriceFeedPercentDifferenceTimes1000 > 1000)
+            if (_maximumPriceFeedPercentDifferenceTimes1000 > 1000)
-                maximumPriceFeedPercentDifferenceTimes1000 -= 500;
+                maximumPriceFeedPercentDifferenceTimes1000 =maximumPriceFeedPercentDifferenceTimes1000 - 500;
            }

		emit MaximumPriceFeedPercentDifferenceChanged(maximumPriceFeedPercentDifferenceTimes1000);
		}
```
- [PriceAggregator.sol#L82](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L82C2-L97C1)
**Poof Of Code**
```diff
File: src/price_feed/PriceAggregator.sol
82: function changePriceFeedModificationCooldown(bool increase) public onlyOwner
		{
+		_priceFeedModificationCooldown = priceFeedModificationCooldown;			
        if (increase)
            {
-            if (priceFeedModificationCooldown < 45 days) //@audit cache priceFeedModificationCooldown
-                priceFeedModificationCooldown += 5 days;
+            if (_priceFeedModificationCooldown < 45 days) 
+                priceFeedModificationCooldown = priceFeedModificationCooldown + 5 days;
            }
        else
            {
-            if (priceFeedModificationCooldown > 30 days)
-                priceFeedModificationCooldown -= 5 days;
+            if (_priceFeedModificationCooldown > 30 days)
+                priceFeedModificationCooldown =priceFeedModificationCooldown - 5 days;
            }

		emit SetPriceFeedCooldownChanged(priceFeedModificationCooldown);
		}
```
- [RewardsConfig.sol#L36](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L36C2-L50C10)
- [RewardsConfig.sol#L52](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L52)
- [RewardsConfig.sol#L70](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L70C7-L77C14)
- [RewardsConfig.sol#L86](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L86C2-L100C10)
- [StakingConfig.sol#L51](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L51C2-L65C10)
- [StakingConfig.sol#L68](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L68C2-L82C10)
- [StakingConfig.sol#L85](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L85C2-L100C6)
- [RewardsConfig.sol#L36](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L36C2-L50C10)
- [RewardsConfig.sol#L52](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L52C2-L66C10)
- [RewardsConfig.sol#L69](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L69C2-L83C10)
- [RewardsConfig.sol#L86](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L86C2-L100C10)

### [G-17] `numberAuthorized()` is called multiple times -> reduce gas costs by avoiding redundant function calls

**Details:**
 the function numberAuthorized() is called multiple times and its result remains constant during the execution of the function, we can optimize the code by caching its result.

 
**Proof of Code:**
- [Airdrop.sol#L56](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L56C4-L70C7)
```solidity
File: src/launch/Airdrop.sol
56:  function allowClaiming() external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
    	require( ! claimingAllowed, "Claiming is already allowed" );
		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

    	// All users receive an equal share of the airdrop.
    	uint256 saltBalance = salt.balanceOf(address(this));
		saltAmountForEachUser = saltBalance / numberAuthorized(); //@audit cache numberAuthorized()

		// Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
		salt.approve( address(staking), saltBalance );

    	claimingAllowed = true;
    	}
```
**Optimized code:**
the result of numberAuthorized() is cached in the variable authorizedCount, and this value is then reused in the subsequent calculations. Caching the result of a function that does not change during the function's execution can help reduce gas costs by avoiding redundant function calls.
```diff
56:  function allowClaiming() external
    	{
    	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
    	require( ! claimingAllowed, "Claiming is already allowed" );
+		// Cache the result of numberAuthorized
+  	    uint256 authorizedCount = numberAuthorized();

-		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+		require(authorizedCount > 0, "No addresses authorized to claim airdrop.");

    	// All users receive an equal share of the airdrop.
    	uint256 saltBalance = salt.balanceOf(address(this));
-		saltAmountForEachUser = saltBalance / numberAuthorized();
+		saltAmountForEachUser = saltBalance / authorizedCount;
		// Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
		salt.approve( address(staking), saltBalance );

    	claimingAllowed = true;
    	}
```
In `Proposals.sol::proposeTokenUnwhitelisting` check multi token to find token whitelist or not. During this check require statement call external function multiple time and result consume more gas. So we can cache multiple external call in local cache and use whenever we needed. 
- [Proposals.sol#L180](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L180C2-L193C1)
**Optimized code:**

```diff
File: src/dao/Proposals.sol
180: function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
+		 address wbtcAddress = address(exchangeConfig.wbtc());
+	    address wethAddress = address(exchangeConfig.weth());
-		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" ); //@audit two external call
+		require( poolsConfig.tokenHasBeenWhitelisted(token, wbtcAddress, wethAddress), "Can only unwhitelist a whitelisted token" );
-		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" ); //@audit
-		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" ); //@audit
+		require( address(token) != address(wbtcAddress), "Cannot unwhitelist WBTC" ); 
+		require( address(token) != address(wethAddress), "Cannot unwhitelist WETH" );
		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );

		string memory ballotName = string.concat("unwhitelist:", Strings.toHexString(address(token)) );
		return _possiblyCreateProposal( ballotName, BallotType.UNWHITELIST_TOKEN, address(token), 0, tokenIconURL, description );
		}
```
### [G-18] Using storage instead of memory for structs/arrays saves gas
**Details:**
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

**Proof of Code:**
- [Proposals.sol#L239](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L239C1-L246C4)
```solidity
File: src/dao/Proposals.sol
239: function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{
		Ballot memory ballot = ballots[ballotID]; //@audit use storage

		// Require that the ballot is actually live
		require( ballot.ballotIsLive, "The specified ballot is not open for voting" );

		// Make sure that the vote type is valid for the given ballot
		if ( ballot.ballotType == BallotType.PARAMETER )
			require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
		else // If a Ballot is not a Parameter Ballot, it is an Approval ballot
			require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );

		// Make sure that the user has voting power before proceeding.
		// Voting power is equal to their userShare of STAKED_SALT.
		// If the user changes their stake after voting they will have to recast their vote.

		uint256 userVotingPower = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
		require( userVotingPower > 0, "Staked SALT required to vote" );

		// Remove any previous votes made by the user on the ballot
@>		UserVote memory lastVote = _lastUserVoteForBallot[ballotID][msg.sender]; //@audit use storage

		// Undo the last vote?
		if ( lastVote.votingPower > 0 )
			_votesCastForBallot[ballotID][lastVote.vote] -= lastVote.votingPower;

		// Update the votes cast for the ballot with the user's current voting power
		_votesCastForBallot[ballotID][vote] += userVotingPower;

		// Remember how the user voted in case they change their vote later
		_lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower );

		emit VoteCast(msg.sender, ballotID, vote, userVotingPower);
		}
```
- [Proposals.sol#L385](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L385C2-L401C1)
**Proof of code:**

```solidity
File: src/dao/Proposals.sol 
385: function canFinalizeBallot( uint256 ballotID ) external view returns (bool) 
		{
@>        Ballot memory ballot = ballots[ballotID]; //@audit 
        if ( ! ballot.ballotIsLive )
        	return false;

        // Check that the minimum duration has passed
        if (block.timestamp < ballot.ballotMinimumEndTime )
            return false;

        // Check that the required quorum has been reached
        if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
            return false;

        return true;
	    }
```
- [Staking.sol#L174](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L174)
**Proof of Code**
```solidity
File: src/staking/Staking.sol
171: function unstakesForUser( address user ) external view returns (Unstake[] memory)
		{
		// Check to see how many unstakes the user has
@>		uint256[] memory unstakeIDs = _userUnstakeIDs[user]; //@audit Use storage
		if ( unstakeIDs.length == 0 )
			return new Unstake[](0);

		// Return them all
		return unstakesForUser( user, 0, unstakeIDs.length - 1 );
		}
```