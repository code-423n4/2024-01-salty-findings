# SALTY GAS OPTIMIZATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."




## [G-01] Refactor `Staking.unstakesForUser()` to avoid coping storage array to memory
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L174

In the `Staking.unstakesForUser()` function as shown below a uint256 array is copied from storage to memory but this array is not used in the function only the length of the array is required in the function so rather than coping the uint256 array from storage to memory just to get the length we can access and cache the length of the uint256 array directly from storage. The diff below shows how the code should be refactored:

```solidity
file: src/staking/Staking.sols

171:	function unstakesForUser( address user ) external view returns (Unstake[] memory)
172:		{
173:		// Check to see how many unstakes the user has
174:		uint256[] memory unstakeIDs = _userUnstakeIDs[user];
175:		if ( unstakeIDs.length == 0 )
176:			return new Unstake[](0);
177:
178:		// Return them all
179:		return unstakesForUser( user, 0, unstakeIDs.length - 1 );
180:		}
```

```diff
diff --git a/src/staking/Staking.sol b/src/staking/Staking.sol
index 22e5970..aa6632d 100644
--- a/src/staking/Staking.sol
+++ b/src/staking/Staking.sol
@@ -171,12 +171,12 @@ contract Staking is IStaking, StakingRewards
        function unstakesForUser( address user ) external view returns (Unstake[] memory)
                {
                // Check to see how many unstakes the user has
-               uint256[] memory unstakeIDs = _userUnstakeIDs[user];
-               if ( unstakeIDs.length == 0 )
+               uint256 unstakeIDsLen = _userUnstakeIDs[user].length;
+               if ( unstakeIDsLen == 0 )
                        return new Unstake[](0);

                // Return them all
-               return unstakesForUser( user, 0, unstakeIDs.length - 1 );
+               return unstakesForUser( user, 0, unstakeIDsLen - 1 );
                }
```


## [G-02] Pre-calculate equations or computations which contain only constant values in constructor
For equations or computations that only involve constants or immutable values they could be computed in the contract's constructor and saved to an immutable variable. Using the immutable variable would be cheaper than re-computing the value every time the function is called.


### 1 Instance
1. #### Compute `keccak256(abi.encodePacked(""))` in the constructor and save value to an immutable variable
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#

Since `abi.encodePacked("")` is a constant value, the value of `keccak256(abi.encodePacked(""))` can be calculated in the constructor. Calling `keccak256(abi.encodePacked(""))` on a known, constant value is a waste of gas rather the calcualtion should be done in the constructor then saved to an immutable variable so that everytime the `proposeWebsiteUpdate()` function is we would not have to re-compute the value of `keccak256(abi.encodePacked(""))` rather it would be replaced with cheaper stack read. The code could be refactored as shown in the diff below:

```solidity
file: src/dao/Proposals.sol

249:	function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
250:		{
251:		require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
252:
253:		string memory ballotName = string.concat("setURL:", newWebsiteURL );
254:		return _possiblyCreateProposal( ballotName, BallotType.SET_WEBSITE_URL, address(0), 0, newWebsiteURL, description );
255:	}
```
```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..beb8039 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -66,6 +66,8 @@ contract Proposals is IProposals, ReentrancyGuard
        // This is to allow some time for users to start staking - as some percent of stake is required to propose ballots and if the total amount staked.
        uint256 immutable firstPossibleProposalTimestamp = block.timestamp + 45 days;

+       bytes32 immutable emptyStringHash;
+

     constructor( IStaking _staking, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IDAOConfig _daoConfig )
                {
@@ -75,6 +77,7 @@ contract Proposals is IProposals, ReentrancyGuard
                daoConfig = _daoConfig;

                salt = exchangeConfig.salt();
+               emptyStringHash = keccak256(abi.encodePacked(""));
         }


@@ -248,7 +251,7 @@ contract Proposals is IProposals, ReentrancyGuard

        function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
                {
-               require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
+               require( keccak256(abi.encodePacked(newWebsiteURL)) != emptyStringHash, "newWebsiteURL cannot be empty" );

                string memory ballotName = string.concat("setURL:", newWebsiteURL );
                return _possiblyCreateProposal( ballotName, BallotType.SET_WEBSITE_URL, address(0), 0, newWebsiteURL, description );
```



## [G-03] Refactor `Airdrop.claimAirdrop()` function to read from storage directly rather than using a function call.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L77

In the `Airdrop.claimAirdrop()` function as shown below rather than having to call the `isAuthorized()` function to check if `msg.sender` is authorized we can read `_authorizedUsers.contains()` diirectly thereby saving the gas cost for 2 `JUMP` and stack setup for the function call. The diff below shows the code should be refactored:

```solidity
file: src/launch/Airdrop.sol

74:     function claimAirdrop() external nonReentrant
75:     	{
76:     	require( claimingAllowed, "Claiming is not allowed yet" );
77:     	require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
78:     	require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );
79:
80: 		// Have the Airdrop contract stake a specified amount of SALT and then transfer it to the user
81: 		staking.stakeSALT( saltAmountForEachUser );
82: 		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );
83: 
84:     	claimed[msg.sender] = true;
85:     	}
```
```diff
diff --git a/src/launch/Airdrop.sol b/src/launch/Airdrop.sol
index 400490a..c1d7e9d 100644
--- a/src/launch/Airdrop.sol
+++ b/src/launch/Airdrop.sol
@@ -74,7 +74,7 @@ contract Airdrop is IAirdrop, ReentrancyGuard
     function claimAirdrop() external nonReentrant
        {
        require( claimingAllowed, "Claiming is not allowed yet" );
-       require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
+       require( _authorizedUsers.contains(msg.sender), "Wallet is not authorized for airdrop" );
        require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );
```
```
Estimated gas saved: 30
```

## [G-04] Redundant state variable getters
Getters for public state variables are automatically generated so there is no need to code
them manually and lose gas.

### 3 Instances
1. #### Make `excludedCountries` mapping variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L384-#L387

The solidity compiler would automatically create a getter function for the `excludedCountries` mapping since it is declared as a `public` variable but a getter function `countryIsExcluded()` was also declared in the contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `excludedCountries` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: src/dao/DAO.sol

67: 	mapping(string=>bool) public excludedCountries;
.
.
.
384:	function countryIsExcluded( string calldata country ) external view returns (bool)
385:		{
386:		return excludedCountries[country];
387:		}
```
```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..7e7bd8b 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -64,7 +64,7 @@ contract DAO is IDAO, Parameters, ReentrancyGuard

        // Countries that have been excluded from access to the DEX (used by AccessManager.sol)
        // Keys as ISO 3166 Alpha-2 Codes
-       mapping(string=>bool) public excludedCountries;
+       mapping(string=>bool) internal excludedCountries;


     constructor( IPools _pools, IProposals _proposals, IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig, IStakingConfig _stakingConfig, IRewardsConfig _rewardsConfig, IStableConfig _stableConfig, IDAOConfig _daoConfig, IPriceAggregator _priceAggregator, IRewardsEmitter _liquidityRewardsEmitter, ICollateralAndLiquidity _collateralAndLiquidity )
```

2. #### Make `ballots` mapping variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L297-#L300

The solidity compiler would automatically create a getter function for the `ballots` mapping since it is declared as a `public` variable but a getter function `ballotForID()` was also declared in the contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `ballots` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: src/dao/Proposals.sol

41:     mapping(uint256=>Ballot) public ballots;
.
.
.
297:	function ballotForID( uint256 ballotID ) external view returns (Ballot memory)
298:		{
299:		return ballots[ballotID];
300:		}
```
```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..e61bbd7 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -38,7 +38,7 @@ contract Proposals is IProposals, ReentrancyGuard
        mapping(string=>uint256) public openBallotsByName;

        // Maps ballotID to the corresponding Ballot
-       mapping(uint256=>Ballot) public ballots;
+       mapping(uint256=>Ballot) internal ballots;
        uint256 public nextBallotID = 1;

        // All of the ballotIDs that are currently open for voting
```

3. #### Make `_arbitrageIndicies` mapping variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L143-#L146

The solidity compiler would automatically create a getter function for the `_arbitrageIndicies` mapping since it is declared as a `public` variable but a getter function `arbitrageIndicies()` was also declared in the contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `_arbitrageIndicies` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: src/pools/PoolStats.sol

24:     mapping(bytes32=>ArbitrageIndicies) public _arbitrageIndicies;
.
.
.
143:	function arbitrageIndicies(bytes32 poolID) external view returns (ArbitrageIndicies memory)
144:		{
145:		return _arbitrageIndicies[poolID];
146:		}
```
```diff
diff --git a/src/pools/PoolStats.sol b/src/pools/PoolStats.sol
index 8e44890..ecd16e2 100644
--- a/src/pools/PoolStats.sol
+++ b/src/pools/PoolStats.sol
@@ -21,7 +21,7 @@ abstract contract PoolStats is IPoolStats
        mapping(bytes32=>uint256) public _arbitrageProfits;

        // Maps poolID(arbToken2, arbToken3) => the indicies (within the whitelistedPools array) of the pools involved in WETH->arbToken2->arbToken3->WETH
-       mapping(bytes32=>ArbitrageIndicies) public _arbitrageIndicies;
+       mapping(bytes32=>ArbitrageIndicies) internal _arbitrageIndicies;


     constructor( IExchangeConfig _exchangeConfig, IPoolsConfig _poolsConfig )
```



## [G-05] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.


### 2 Instances

1. #### Perform the external call `stableConfig.minimumCollateralRatioPercent()` outside the loop and cache the result.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326

The external call `stableConfig.minimumCollateralRatioPercent()` should be made outside the loop and the result cached since the value it returns is not dependent on the loop iterations. Implementing this change would help save at least `100` gas units per iteration of the loop. The diff below shows how the code should be refactored:

```solidity
file: src/stable/CollateralAndLiquidity.sol

311:	function findLiquidatableUsers( uint256 startIndex, uint256 endIndex ) public view returns (address[] memory)
312:		{
313:		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);
.
.
.
320:		if ( totalCollateralValue != 0 )
321:			for ( uint256 i = startIndex; i <= endIndex; i++ )
322:				{
323:				address wallet = _walletsWithBorrowedUSDS.at(i);
324:
325:				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
326:				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;    //@audit cache stableConfig.minimumCollateralRatioPercent() outside loop
327:
328:				// Determine minCollateral in terms of minCollateralValue
329:				uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
330:
331:				// Make sure the user has at least minCollateral
332:				if ( userShareForPool( wallet, collateralPoolID ) < minCollateral )
333:					liquidatableUsers[count++] = wallet;
334:				}
335:
336:		// Resize the array to match the actual number of liquidatable positions found
337:		address[] memory resizedLiquidatableUsers = new address[](count);
338:		for ( uint256 i = 0; i < count; i++ )
339:			resizedLiquidatableUsers[i] = liquidatableUsers[i];
340:
341:		return resizedLiquidatableUsers;
342:		}
```
```diff
diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
index fc14695..6d35494 100644
--- a/src/stable/CollateralAndLiquidity.sol
+++ b/src/stable/CollateralAndLiquidity.sol
@@ -316,14 +316,14 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                // Cache
                uint256 totalCollateralShares = totalShares[collateralPoolID];
                uint256 totalCollateralValue = totalCollateralValueInUSD();
-
+               uint256 _minimumCollateralRatioPercent = stableConfig.minimumCollateralRatioPercent();
                if ( totalCollateralValue != 0 )
                        for ( uint256 i = startIndex; i <= endIndex; i++ )
                                {
                                address wallet = _walletsWithBorrowedUSDS.at(i);

                                // Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * _minimumCollateralRatioPercent) / 100;

                                // Determine minCollateral in terms of minCollateralValue
                                uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
```
```
Estimated gas saved: 100 gas units per iteration
```


2. #### Perform the external call `_openBallotsForTokenWhitelisting.length()` outside the loop and cache the result.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423

The external call `_openBallotsForTokenWhitelisting.length()` should be made outside the loop and the result cached since the value it returns is not dependent on the loop iterations. Implementing this change would help save at least `100` gas units per iteration of the loop. The diff below shows how the code should be refcatored:

```solidity
file: src/dao/Proposals.sol

417:	function tokenWhitelistingBallotWithTheMostVotes() external view returns (uint256)
418:		{
419:		uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN);
420:
421:		uint256 bestID = 0;
422:		uint256 mostYes = 0;
423:		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )    //@audit cache external call oustide loop
424:			{
425:			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
426:			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
427:			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];
428:
429:			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
430:			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
431:			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
432:				{
433:				bestID = ballotID;
434:				mostYes = yesTotal;
435:				}
436:			}
437:
438:		return bestID;
439:		}
```
```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..59c4208 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -420,7 +420,8 @@ contract Proposals is IProposals, ReentrancyGuard

                uint256 bestID = 0;
                uint256 mostYes = 0;
-               for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
+               uint len = _openBallotsForTokenWhitelisting.length();
+               for( uint256 i = 0; i < len; i++ )
                        {
                        uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
                        uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
```
```
Estimated gas saved: 100 gas units per iteration
```

## [G-06] Refactor functions to avoid unnecessary external calls

### Instances

1. #### Refactor `DAO._finalizeTokenWhitelisting()` to avoid unnecessary external calls
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L246-#L266

In the `DAO._finalizeTokenWhitelisting()` function as shown below rather than having to perform the external calls `exchangeConfig.wbtc()` and `exchangeConfig.weth()` twice in the function we can make the external call once, cache the returned value to a stack variable then read from the stack variable subsequently. In implementing this change we would save above `100` gas unit for every subsequent external call. The diff below shows how the code should be refactored:

```solidity
file: src/dao/DAO.sol

235:	function _finalizeTokenWhitelisting( uint256 ballotID ) internal
236:		{
237:		if ( proposals.ballotIsApproved(ballotID ) )
238:			{
.
.
.
253:			// All tokens are paired with both WBTC and WETH, so whitelist both pairings
254:			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255:			poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );
256:
257:			bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
258:			bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
.
.
.
273:		}
```

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..3168a99 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -251,11 +251,14 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

                        // All tokens are paired with both WBTC and WETH, so whitelist both pairings
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );
+                       IERC20 _wbtc = exchangeConfig.wbtc();
+                       IERC20 _weth = exchangeConfig.weth();

-                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), _wbtc );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1),_weth );
+
+                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), _wbtc );
+                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), _weth );

                        // Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
                        AddedReward[] memory addedRewards = new AddedReward[](2);
```
```
Estimated gas saved: 200 gas units
```

2. #### Refactor `DAO._executeApproval()` to avoid unnecessary external calls
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L170-#L172

In the `DAO._executeApproval()` function as shown below rather than having to perform the external call `exchangeConfig.salt()` on multiple occasions in the function we can make the external call once, cache the returned value to a stack variable then read from the stack variable subsequently. In implementing this change we would save above `100` gas unit for every subsequent external call. The diff below shows how the code should be refactored:

```solidity
file: src/dao/DAO.sol

155:	function _executeApproval( Ballot memory ballot ) internal
156:		{
157:		if ( ballot.ballotType == BallotType.UNWHITELIST_TOKEN )
158:			{
159:			// All tokens are paired with both WBTC and WETH so unwhitelist those pools
160:			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
161:			poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );
162:
163:			emit UnwhitelistToken(IERC20(ballot.address1));
164:			}
165:
166:		else if ( ballot.ballotType == BallotType.SEND_SALT )
167:			{
168:			// Make sure the contract has the SALT balance before trying to send it.
169:			// This should not happen but is here just in case - to prevent approved proposals from reverting on finalization.
170:			if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 ) //@audit cache exchangeConfig.salt()
171:				{
172:				IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );
173:
174:				emit SaltSent(ballot.address1, ballot.number1);
175:				}
176:			}
.
.
.
		}
```
```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..de9d2bb 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -167,9 +167,10 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        {
                        // Make sure the contract has the SALT balance before trying to send it.
                        // This should not happen but is here just in case - to prevent approved proposals from reverting on finalization.
-                       if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
+                       ISalt _salt = exchangeConfig.salt();
+                       if ( _salt.balanceOf(address(this)) >= ballot.number1 )
                                {
-                               IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );
+                               IERC20(_salt).safeTransfer( ballot.address1, ballot.number1 );

                                emit SaltSent(ballot.address1, ballot.number1);
                                }
```
```
Estimated gas saved: 100 gas units
```

3. #### Refactor `Proposals.proposeTokenUnwhitelisting()` to avoid unnecessary external calls
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L180-#L191

In the `Proposals.proposeTokenUnwhitelisting()` function as shown below rather than having to perform the external calls `exchangeConfig.wbtc()` and `exchangeConfig.weth()` twice in the function we can make the external calls once, cache the returned value to a stack variable then read from the stack variable subsequently. In implementing this change we would save above `100` gas unit for every subsequent external call. The diff below shows how the code should be refactored:

```solidity
file: src/dao/Proposals.sol

180:	function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181:		{
182:		require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
183:		require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
184:		require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
185:		require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
186:		require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
187:		require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
188:
189:		string memory ballotName = string.concat("unwhitelist:", Strings.toHexString(address(token)) );
190:		return _possiblyCreateProposal( ballotName, BallotType.UNWHITELIST_TOKEN, address(token), 0, tokenIconURL, description );
191:		}
```

```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..3c0f84d 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -179,9 +179,12 @@ contract Proposals is IProposals, ReentrancyGuard

        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
                {
-               require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" ); -               require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
-               require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
+               IERC20 _wbtc = exchangeConfig.wbtc();
+               IERC20 _weth = exchangeConfig.weth();
+
+               require( poolsConfig.tokenHasBeenWhitelisted(token, _wbtc, _weth), "Can only unwhitelist a whitelisted token" );
+               require( address(token) != address(_wbtc), "Cannot unwhitelist WBTC" );
+               require( address(token) != address(_weth), "Cannot unwhitelist WETH" );
                require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
                require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
                require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
```
```
Estimated gas saved: 200 gas units
```

4. #### Refactor `Upkeep.step11()` to avoid unnecessary external calls
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L231-#L239

In the `Upkeep.step11()` function as shown below rather than having to perform the external call `exchangeConfig.teamVestingWallet()` on twice in the function we can make the external call once, cache the returned value to a stack variable then read from the stack variable subsequently. In implementing this change we would save above `100` gas unit for every subsequent external call. The diff below shows how the code should be refactored:

```solidity
file: src/Upkeep.sol

231:	function step11() public onlySameContract
232:		{
233:		uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
234:
235:		// teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
236:		VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
237:
238:		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
239:		}
```

```diff
diff --git a/src/Upkeep.sol b/src/Upkeep.sol
index 7587bde..77c3f88 100644
--- a/src/Upkeep.sol
+++ b/src/Upkeep.sol
@@ -230,10 +230,11 @@ contract Upkeep is IUpkeep, ReentrancyGuard
        // 11. Sends SALT from the team vesting wallet to the team (linear distribution over 10 years).
        function step11() public onlySameContract
                {
-               uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
+               VestingWallet _vestingWallet = exchangeConfig.teamVestingWallet();
+               uint256 releaseableAmount = VestingWallet(payable(_vestingWallet)).releasable(address(salt));

                // teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
-               VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
+               VestingWallet(payable(_vestingWallet)).release(address(salt));

                salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
                }
```
```
Estimated gas saved: 100 gas units
```





## [G-07] Avoid Reading and writing to state if amount is zero

### Instances

1. #### Refactor `Liquidizer.incrementBurnableUSDS()` to avoid state read/write if `usdsToBurn` is zero
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L81-#L88

In the `Liquidizer.incrementBurnableUSDS()` function as shown below checks should be implemented to avoid reading and writing to state if the `usdsToBurn` argument is zero this is because if `usdsToBurn` is 0 the statement `usdsThatShouldBeBurned += usdsToBurn` would not change the value of `usdsThatShouldBeBurned` since its being incremented by zero. This then means that in scenarios where `usdsToBurn` is 0 the statement `usdsThatShouldBeBurned += usdsToBurn` is re-assigning the same value to state i.e there is no state change. Implementing this would help save `5000` gas units in this scenario. The diff below shows how the function should be refactored:

```solidity
file: src/stable/Liquidizer.sol

81: 	function incrementBurnableUSDS( uint256 usdsToBurn ) external
82: 		{
83: 		require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
84:  
85: 		usdsThatShouldBeBurned += usdsToBurn;
86:  
87: 		emit incrementedBurnableUSDS(usdsThatShouldBeBurned);
88: 		}
```

```diff
diff --git a/src/stable/Liquidizer.sol b/src/stable/Liquidizer.sol
index 64636f1..ef8aaa3 100644
--- a/src/stable/Liquidizer.sol
+++ b/src/stable/Liquidizer.sol
@@ -80,11 +80,14 @@ contract Liquidizer is ILiquidizer, Ownable
        // Called when a user's collateral position has been liquidated - to indicate that the borrowed USDS from that position needs to be burned.
        function incrementBurnableUSDS( uint256 usdsToBurn ) external
                {
-               require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
+               if (usdsToBurn > 0){
+                       require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );

-               usdsThatShouldBeBurned += usdsToBurn;
+                       usdsThatShouldBeBurned += usdsToBurn;
+
+                       emit incrementedBurnableUSDS(usdsThatShouldBeBurned);
+               }

-               emit incrementedBurnableUSDS(usdsThatShouldBeBurned);
                }
```
```
Estimated gas saved: 5000 gas units.
```

2. #### Refactor `CollateralAndLiquidity.borrowUSDS()` to avoid state read/write if `amountBorrowed` is zero
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L95-#L111

In the `CollateralAndLiquidity.borrowUSDS()` function as shown below checks should be implemented to avoid reading and writing to state if the `amountBorrowed` argument is zero this is because if `amountBorrowed` is 0 the statement `usdsBorrowedByUsers[msg.sender] += amountBorrowed` would not change the value of `usdsBorrowedByUsers[msg.sender]` since its being incremented by zero. This then means that in scenarios where `amountBorrowed` is 0 the statement `usdsBorrowedByUsers[msg.sender] += amountBorrowed` is re-assigning the same value to state i.e there is no state change. Implementing this would help save above `5000` gas units in this scenario. The diff below shows how the function should be refactored:

```solidity
file: src/stable/CollateralAndLiquidity.sol

95:     function borrowUSDS( uint256 amountBorrowed ) external nonReentrant
96: 		{
97: 		require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
98: 		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
99: 		require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );
100:
101:  		// Increase the borrowed amount for the user
102:		usdsBorrowedByUsers[msg.sender] += amountBorrowed;
103:
104:		// Remember that the user has borrowed USDS (so they can later be checked for sufficient collateralization ratios and liquidated if necessary)
105:		_walletsWithBorrowedUSDS.add(msg.sender);
106:
107:		// Mint USDS and send it to the user
108:		usds.mintTo( msg.sender, amountBorrowed );
109:
110:		emit BorrowedUSDS(msg.sender, amountBorrowed);
111:		}
```
```diff
diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
index fc14695..4dab96d 100644
--- a/src/stable/CollateralAndLiquidity.sol
+++ b/src/stable/CollateralAndLiquidity.sol
@@ -94,20 +94,23 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
        // Requires exchange access for the sending wallet
     function borrowUSDS( uint256 amountBorrowed ) external nonReentrant
                {
-               require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
-               require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
-               require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );
+               if (amountBorrowed > 0) {
+                       require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
+                       require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
+                       require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );
+
+                       // Increase the borrowed amount for the user
+                       usdsBorrowedByUsers[msg.sender] += amountBorrowed;

-               // Increase the borrowed amount for the user
-               usdsBorrowedByUsers[msg.sender] += amountBorrowed;
+                       // Remember that the user has borrowed USDS (so they can later be checked for sufficient collateralization ratios and liquidated if necessary)
+                       _walletsWithBorrowedUSDS.add(msg.sender);

-               // Remember that the user has borrowed USDS (so they can later be checked for sufficient collateralization ratios and liquidated if necessary)
-               _walletsWithBorrowedUSDS.add(msg.sender);
+                       // Mint USDS and send it to the user
+                       usds.mintTo( msg.sender, amountBorrowed );

-               // Mint USDS and send it to the user
-               usds.mintTo( msg.sender, amountBorrowed );
+                       emit BorrowedUSDS(msg.sender, amountBorrowed);

-               emit BorrowedUSDS(msg.sender, amountBorrowed);
+               }
                }
```
```
Estimated gas saved: 5000
```




## [G-08] Refactor `Airdrop.allowClaiming()` function to read from storage directly rather than using a function call.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L60
- https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L64

In the `Airdrop.allowClaiming()` function as shown below rather than having to call the `numberAuthorized()` function to determine if `_authorizedUsers.length()` is greater than 0 we can read the value `_authorizedUsers.length()` diirectly thereby saving the gas cost for 2 `JUMP` and stack setup for the two times the `numberAuthorized()` function is called. The diff below shows the code should be refactored:

```solidity
file: src/launch/Airdrop.sol

56:     function allowClaiming() external
57:     	{
58:     	require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
59:     	require( ! claimingAllowed, "Claiming is already allowed" );
60: 		require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
61:
62:     	// All users receive an equal share of the airdrop.
63:     	uint256 saltBalance = salt.balanceOf(address(this));
64: 		saltAmountForEachUser = saltBalance / numberAuthorized();
65:
66: 		// Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
67: 		salt.approve( address(staking), saltBalance );
68:
69:     	claimingAllowed = true;
70:     	}
```
```diff
diff --git a/src/launch/Airdrop.sol b/src/launch/Airdrop.sol
index 400490a..472a50a 100644
--- a/src/launch/Airdrop.sol
+++ b/src/launch/Airdrop.sol
@@ -55,13 +55,14 @@ contract Airdrop is IAirdrop, ReentrancyGuard
        // Called by the InitialDistribution contract during its distributionApproved() function - which is called on successful conclusion of the BootstrappingBallot
     function allowClaiming() external
        {
+               uint256 _authorizedUsersLen = _authorizedUsers.length();
        require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
        require( ! claimingAllowed, "Claiming is already allowed" );
-               require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+               require(_authorizedUsersLen > 0, "No addresses authorized to claim airdrop.");

        // All users receive an equal share of the airdrop.
        uint256 saltBalance = salt.balanceOf(address(this));
-               saltAmountForEachUser = saltBalance / numberAuthorized();
+               saltAmountForEachUser = saltBalance / _authorizedUsersLen;

                // Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
                salt.approve( address(staking), saltBalance );
```
```
Estimated gas saved: 60 gas units
```




## [G-09] Refactor external/internal function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the external/internal functions to pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the internal functions.

### 1 Instances
1. #### Refactor external function `finalizeBallot()` and internal functions `_finalizeParameterBallot()`, `_finalizeTokenWhitelisting()` and `_finalizeApprovalBallot()` to avoid unnecessary external calls.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L278-#L291

The `finalizeBallot()` function as shown below performs an external call `proposals.ballotForID(ballotID)` which returns a value of type `Ballot memory`. The `finalizeBallot()` function invokes 3 internal functions `_finalizeParameterBallot()`, `_finalizeTokenWhitelisting()` and `_finalizeApprovalBallot()`  (note that these functions were only invoked in the `finalizeBallot()` function) these internal functions also perform the same external call `proposals.ballotForID(ballotID)`
which is redundant and a waste of gas rather we should refactor the external function `finalizeBallot()` and the internal functions `_finalizeParameterBallot()`, `_finalizeTokenWhitelisting()` and `_finalizeApprovalBallot()` such that result of the `proposals.ballotForID(ballotID)` external call made in the `finalizeBallot()` function is cached and passed as an argument to the internal functions. The diff below show how the functions should be refactored:

```solidity
file: src/dao/DAO.sol

113:	function _finalizeParameterBallot( uint256 ballotID ) internal
114:		{
115:		Ballot memory ballot = proposals.ballotForID(ballotID);  //@audit proposals.ballotForID(ballotID) external call made
116:
117:		Vote winningVote = proposals.winningParameterVote(ballotID);
118:
119:		if ( winningVote == Vote.INCREASE )
120:			_executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
121:		else if ( winningVote == Vote.DECREASE )
122:			_executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
123:
124:		// Finalize the ballot even if NO_CHANGE won
125:		proposals.markBallotAsFinalized(ballotID);
126:
127:		emit BallotFinalized(ballotID, winningVote);
128:		}
.
.
.
219:	function _finalizeApprovalBallot( uint256 ballotID ) internal
220:		{
221:		if ( proposals.ballotIsApproved(ballotID ) )
222:			{
223:			Ballot memory ballot = proposals.ballotForID(ballotID); //@audit proposals.ballotForID(ballotID) external call made
224:			_executeApproval( ballot );
225:			}
226:
227:		proposals.markBallotAsFinalized(ballotID);
228:		}
.
.
.
235:	function _finalizeTokenWhitelisting( uint256 ballotID ) internal
236:		{
237:		if ( proposals.ballotIsApproved(ballotID ) )
238:			{
239:			// The ballot is approved. Any reversions below will allow the ballot to be attemped to be finalized later - as the ballot won't be finalized on reversion.
240:			Ballot memory ballot = proposals.ballotForID(ballotID); //@audit proposals.ballotForID(ballotID) external call made
241:
242:			uint256 bootstrappingRewards = daoConfig.bootstrappingRewards();
.
.
.
271:		// Mark the ballot as finalized (which will also remove it from the list of open token whitelisting proposals)
272:		proposals.markBallotAsFinalized(ballotID);
273:		}
.
.
.
278:	function finalizeBallot( uint256 ballotID ) external nonReentrant
279:		{
280:		// Checks that ballot is live, and minimumEndTime and quorum have both been reached
281:		require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );
282:
283:		Ballot memory ballot = proposals.ballotForID(ballotID); //@audit proposals.ballotForID(ballotID) external call made
284:
285:		if ( ballot.ballotType == BallotType.PARAMETER )
286:			_finalizeParameterBallot(ballotID);
287:		else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
288:			_finalizeTokenWhitelisting(ballotID);
289:		else
290:			_finalizeApprovalBallot(ballotID);
291:		}
```

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..4c664be 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -110,9 +110,8 @@ contract DAO is IDAO, Parameters, ReentrancyGuard


        // Finalize the vote for a parameter ballot (increase, decrease or no_change) for a given parameter
-       function _finalizeParameterBallot( uint256 ballotID ) internal
+       function _finalizeParameterBallot( uint256 ballotID, Ballot memory ballot ) internal
                {
-               Ballot memory ballot = proposals.ballotForID(ballotID);

                Vote winningVote = proposals.winningParameterVote(ballotID);

@@ -216,11 +215,11 @@ contract DAO is IDAO, Parameters, ReentrancyGuard


        // Finalize the vote for an approval ballot (yes or no) for a given proposal
-       function _finalizeApprovalBallot( uint256 ballotID ) internal
+       function _finalizeApprovalBallot( uint256 ballotID, Ballot memory ballot ) internal
                {
                if ( proposals.ballotIsApproved(ballotID ) )
                        {
-                       Ballot memory ballot = proposals.ballotForID(ballotID);
+
                        _executeApproval( ballot );
                        }

@@ -232,12 +231,12 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
        // If the proposal is currently the whitelisting proposal with the most yes votes then the token can be whitelisted.
        // Only the top voted whitelisting proposal can be finalized - as whitelisting requires bootstrapping rewards to be sent from the DAO.
        // If NO > YES than the proposal is removed immediately (quorum would already have been determined - in canFinalizeBallot as called from finalizeBallot).-       function _finalizeTokenWhitelisting( uint256 ballotID ) internal
+       function _finalizeTokenWhitelisting( uint256 ballotID, Ballot memory ballot ) internal
                {
                if ( proposals.ballotIsApproved(ballotID ) )
                        {
                        // The ballot is approved. Any reversions below will allow the ballot to be attemped to be finalized later - as the ballot won't be finalized on reversion.
-                       Ballot memory ballot = proposals.ballotForID(ballotID);
                        uint256 bootstrappingRewards = daoConfig.bootstrappingRewards();

@@ -283,11 +282,11 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                Ballot memory ballot = proposals.ballotForID(ballotID);

                if ( ballot.ballotType == BallotType.PARAMETER )
-                       _finalizeParameterBallot(ballotID);
+                       _finalizeParameterBallot(ballotID, ballot);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
-                       _finalizeTokenWhitelisting(ballotID);
+                       _finalizeTokenWhitelisting(ballotID, ballot);
                else
-                       _finalizeApprovalBallot(ballotID);
+                       _finalizeApprovalBallot(ballotID, ballot);
                }
```


## [G-10] Refactor external/internal function to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the external/internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

### 1 Instances
1. #### Refactor `grantAccess()` and `_verifyAccess()` functions to avoid unnecessary SLOAD
- https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L51-#L68

The `grantAccess()` function as shown below invokes the `_verifyAccess()` but both function reads the `geoVersion` state variable and the `_verifyAccess()` function is only called in the `grantAccess()` function. We can refactor both functions such that the `grantAccess()` function passes the `geoVersion` as an argument to the `_verifyAccess()` function since this function just reads and do not modify the `geoVersion` state variable. Refactoring the code this way would save 1 `SLOAD` 100 gas units (cold access). The diff below shows how the functions could be modified: 

```solidity
file: src/AccessManager.sol

51:    function _verifyAccess(address wallet, bytes memory signature ) internal view returns (bool)
52:    	{
53:		bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
54:
55:		return SigningTools._verifySignature(messageHash, signature);
56:    	}
.
.
.
61:    function grantAccess(bytes calldata signature) external
62:    	{
63:    	require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );
64:
65:        _walletsWithAccess[geoVersion][msg.sender] = true;
66:
67:        emit AccessGranted( msg.sender, geoVersion );
68:    	}
```

```diff
diff --git a/src/AccessManager.sol b/src/AccessManager.sol
index a9b0a50..c54a157 100644
--- a/src/AccessManager.sol
+++ b/src/AccessManager.sol
@@ -48,9 +48,9 @@ contract AccessManager is IAccessManager

        // Verify that the access request was signed by the authoratative signer.
        // Note that this is only a simplistic default mechanism and can be changed by the DAO at any time (either altering the regional restrictions themselves or replacing the access mechanism entirely).
-    function _verifyAccess(address wallet, bytes memory signature ) internal view returns (bool)
+    function _verifyAccess(address wallet, bytes memory signature, uint256 _geoVersion ) internal view returns (bool)
        {
-               bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
+               bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, _geoVersion, wallet));

                return SigningTools._verifySignature(messageHash, signature);
        }
@@ -60,11 +60,12 @@ contract AccessManager is IAccessManager
        // Requires the accompanying correct message signature from the offchain verifier.
     function grantAccess(bytes calldata signature) external
        {
-       require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );
+                       uint256 _geoVersion = geoVersion;
+       require( _verifyAccess(msg.sender, signature, _geoVersion), "Incorrect AccessManager.grantAccess signatory" );

-        _walletsWithAccess[geoVersion][msg.sender] = true;
+        _walletsWithAccess[_geoVersion][msg.sender] = true;

-        emit AccessGranted( msg.sender, geoVersion );
+        emit AccessGranted( msg.sender, _geoVersion );
        }
```





## [G-11] Refactor `CollateralAndLiquidity.findLiquidatableUsers()` function to read from storage directly rather than using a function call.
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L347
https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L350

In the `CollateralAndLiquidity.findLiquidatableUsers()` function as shown below rather than having to call the `numberAuthorized()` function to determine if `_walletsWithBorrowedUSDS.length()` is equal to 0 we can read the value `_walletsWithBorrowedUSDS.length()` diirectly thereby saving the gas cost for 2 `JUMP` and stack setup for the two times the `numberAuthorized()` function is called. The diff below shows the code should be refactored:

```solidity
file: src/stable/CollateralAndLiquidity.sol

345:	function findLiquidatableUsers() external view returns (address[] memory)
346:		{
347:		if ( numberOfUsersWithBorrowedUSDS() == 0 )
348:			return new address[](0);
349:
350:		return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
351:		}
```

```diff
diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
index fc14695..5dc1796 100644
--- a/src/stable/CollateralAndLiquidity.sol
+++ b/src/stable/CollateralAndLiquidity.sol
@@ -344,9 +344,10 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity

        function findLiquidatableUsers() external view returns (address[] memory)
                {
-               if ( numberOfUsersWithBorrowedUSDS() == 0 )
+               uint256 _walletsWithBorrowedUSDSLen = _walletsWithBorrowedUSDS.length();
+               if ( _walletsWithBorrowedUSDSLen == 0 )
                        return new address[](0);

-               return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
+               return findLiquidatableUsers( 0, _walletsWithBorrowedUSDSLen - 1 );
                }
        }
```
```
Estimated gas saved: 60 gas units
```



## [G-12] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state


### Instances

1. #### Use stack variable `_pools` in place of `pools`.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L71-#L73

In the `setContracts()` function as shown below we could use the parameter `_pools` in place of `pools` in the following statements `wbtc.approve( address(pools), type(uint256).max )`, `weth.approve( address(pools), type(uint256).max )` and `dai.approve( address(pools), type(uint256).max )` to avoid the gas expensive operation of reading from state. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD Gcoldaccess` `2100` gas units and 2 `SLOAD Gwarmaccess` `100` gas units. The diff below shows how the code should be refactored:

```solidity
file: src/stable/Liquidizer.sol

63: 	function setContracts( ICollateralAndLiquidity _collateralAndLiquidity, IPools _pools, IDAO _dao) external onlyOwner
64: 		{
65: 		collateralAndLiquidity = _collateralAndLiquidity;
66: 		pools = _pools;
67: 		dao = _dao;
68: 
69: 		// Gas saving approve for future swaps.
70: 		// Normally, this contract will have zero balances of these tokens.
71: 		wbtc.approve( address(pools), type(uint256).max );  //@audit use _pools in place of pools
72: 		weth.approve( address(pools), type(uint256).max );  //@audit use _pools in place of pools
73: 		dai.approve( address(pools), type(uint256).max );  //@audit use _pools in place of pools
74:
75: 		// setContracts can only be called once
76: 		renounceOwnership();
77: 		}
```

```diff
diff --git a/src/stable/Liquidizer.sol b/src/stable/Liquidizer.sol
index 64636f1..7256fee 100644
--- a/src/stable/Liquidizer.sol
+++ b/src/stable/Liquidizer.sol
@@ -68,9 +68,9 @@ contract Liquidizer is ILiquidizer, Ownable

                // Gas saving approve for future swaps.
                // Normally, this contract will have zero balances of these tokens.
-               wbtc.approve( address(pools), type(uint256).max );
-               weth.approve( address(pools), type(uint256).max );
-               dai.approve( address(pools), type(uint256).max );
+               wbtc.approve( address(_pools), type(uint256).max );
+               weth.approve( address(_pools), type(uint256).max );
+               dai.approve( address(_pools), type(uint256).max );

                // setContracts can only be called once
                renounceOwnership();
```
```
Estimated gas saved: 2294 gas units.
```


2. #### Use stack variables `_proposedMainWallet` and `_proposedConfirmationWallet` in place of `proposedMainWallet` and `proposedConfirmationWallet` respectively.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L54

In the `proposeWallets()` function as shown below we could use the parameters `_proposedMainWallet` and `_proposedConfirmationWallet` in place of `proposedMainWallet` and `proposedConfirmationWallet` in the emit statement `emit WalletProposal(proposedMainWallet, proposedConfirmationWallet)` to avoid the gas expensive operation of reading from state. In implementing this change a cheaper stack read `3 gas` would be used in place of 2 `SLOAD Gcoldaccess` `2100` gas units . The diff below shows how the code should be refactored:

```solidity
file: src/ManagedWallet.sol

42: 	function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
43: 		{
44: 		require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
45: 		require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
46: 		require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
47:
48: 		// Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
49: 		require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );
50:
51: 		proposedMainWallet = _proposedMainWallet;
52: 		proposedConfirmationWallet = _proposedConfirmationWallet;
53:
54: 		emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
55: 		}
```
```diff
diff --git a/src/ManagedWallet.sol b/src/ManagedWallet.sol
index 7082030..1597ccf 100644
--- a/src/ManagedWallet.sol
+++ b/src/ManagedWallet.sol
@@ -51,7 +51,7 @@ contract ManagedWallet is IManagedWallet
                proposedMainWallet = _proposedMainWallet;
                proposedConfirmationWallet = _proposedConfirmationWallet;

-               emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
+               emit WalletProposal(_proposedMainWallet, _proposedConfirmationWallet);
                }
```
```
Estimated gas saved: 4200 gas units
```



3. #### Use global variable `msg.sender` in place of `proposedMainWallet`.
- https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L80

In the `changeWallets()` function as shown below we could use the global variable `msg.sender` in place of `proposedMainWallet` in the statement `mainWallet = proposedMainWallet` statement to avoid the gas expensive operation of reading from state as the require statement `require( msg.sender == proposedMainWallet, "Invalid sender" )` ensures that `msg.sender` is equal `proposedMainWallet`. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD Gcoldaccess` `2100` gas units . The diff below shows how the code should be refactored:

```solidity
file: src/ManagedWallet.sol

73: 	function changeWallets() external
74: 		{
75: 		// proposedMainWallet calls the function - to make sure it is a valid address.
76: 		require( msg.sender == proposedMainWallet, "Invalid sender" );
77: 		require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
78:
79: 		// Set the wallets
80: 		mainWallet = proposedMainWallet;
81: 		confirmationWallet = proposedConfirmationWallet;
82: 
83: 		emit WalletChange(mainWallet, confirmationWallet);
84:
85: 		// Reset
86: 		activeTimelock = type(uint256).max;
87: 		proposedMainWallet = address(0);
88: 		proposedConfirmationWallet = address(0);
89: 		}
```
```diff
diff --git a/src/ManagedWallet.sol b/src/ManagedWallet.sol
index 7082030..94f1308 100644
--- a/src/ManagedWallet.sol
+++ b/src/ManagedWallet.sol
@@ -77,7 +77,7 @@ contract ManagedWallet is IManagedWallet
                require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

                // Set the wallets
-               mainWallet = proposedMainWallet;
+               mainWallet = msg.sender;
                confirmationWallet = proposedConfirmationWallet;

                emit WalletChange(mainWallet, confirmationWallet);
```
```
Estimated gas saved: 2100
```




## [G-13] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.

#### Please note this instances were not included in the bots reports

### Instances

1. #### The calculation `uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions` can be unchecked
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L186

In the `step6()` function it is not possible that the computation `block.timestamp - lastUpkeepTimeEmissions` statement would underflow as the value of `block.timestamp` would always be greater than that of `lastUpkeepTimeEmissions` so we can perform the `uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions` statement in an `unchecked {}` block as this would save some gas from the unnecessary internal underflow checks.

```solidity
file: src/Upkeep.sol

184:	function step6() public onlySameContract
185:		{
186:		uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions;    //@audit can be unchecked
187:		emissions.performUpkeep(timeSinceLastUpkeep);
188:
189:		lastUpkeepTimeEmissions = block.timestamp;
190:		}
```


2. #### The calculation `uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeRewardsEmitters` can be unchecked
- https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L207

In the `step8()` function it is not possible that the computation `block.timestamp - lastUpkeepTimeRewardsEmitters` statement would underflow as the value of `block.timestamp` would always be greater than that of `lastUpkeepTimeRewardsEmitters` so we can perform the `uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeRewardsEmitters` statement in an `unchecked {}` block as this would save some gas from the unnecessary internal underflow checks.

```solidity
file: src/Upkeep.sol

205:	function step8() public onlySameContract
206:		{
207:		uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeRewardsEmitters;  //@audit can be unchecked
208:
209:		saltRewards.stakingRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
210:		saltRewards.liquidityRewardsEmitter().performUpkeep(timeSinceLastUpkeep);
211:
212:		lastUpkeepTimeRewardsEmitters = block.timestamp;
213:		}
```


## [G-14] Assigning state variables directly with named struct constructors wastes gas
Using named arguments for struct means that the compiler needs to organize the fields in memory before doing the assignment, which wastes gas. Set each field directly in storage (use dot-notation), or use the unnamed version of the constructor.

### Proof of concept
```solidity
struct Addresses {
    address address1;
    address address2;
    address address3;
}


contract UsingNamedStruct {

    Addresses public myAddresses;

    function setMyAddresses() public {
        myAddresses = Addresses({
            address1: address(1),
            address2: address(1),
            address3: address(1)
        });
    }
}
```
```
test/UsingNamedStruct.t.sol:UsingNamedStructTest
[PASS] test_setMyAddresses() (gas: 71812)
```

```solidity
struct Addresses {
    address address1;
    address address2;
    address address3;
}

contract SettingStorageFields {

    Addresses public myAddresses;


    function setMyAddresses() public {
        myAddresses.address1 = address(1);
        myAddresses.address2 = address(1);
        myAddresses.address3 = address(1);
    }
    
}
```
```
 test/SettingStorageField.t.sol:SettingStorageFieldsTest
[PASS] test_setMyAddresses() (gas: 71722)
```

### Instances
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L109
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L54




## [G-15] memory variable should created outside of loop
memory variable should created outside of loop, and get overriden with each iteration of loop, By doing so we save gas cost for memory variable creation in each iteration.

### Proof of Concept
```solidity
contract VarInLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        for(uint256 i; i < len; ++i) {
            uint256 doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarInLoop.t.sol:VarInLoopTest
[PASS] test_doubleNumbers() (gas: 38185)
```


```
contract VarOutLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        uint256 doubleNum;
        for(uint256 i; i < len; ++i) {
            doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarOutLoop.t.sol:VarOutLoopTest
[PASS] test_doubleNumbers() (gas: 38170)
```

### 25 Instances
- https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L117
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L425
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L426
- https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L427
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L84
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L89
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L90
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L91
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L94
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L109
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L112
- https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L115
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L118
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L121
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L66
- https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L67
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L323
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326
- https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L329
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L154
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L156
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L187
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L189
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L192
- https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L298






## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.
