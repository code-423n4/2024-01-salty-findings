## QA Summary<a name="QA Summary">

### QA Issues
| |Issue|Contexts|
|-|:-|:-:|
| [QA&#x2011;1](#QA&#x2011;1) | `address wallet` in `USDS.mintTo()` can be `address(collateralAndLiquidity)` | 1 |
| [QA&#x2011;2](#QA&#x2011;2) | `ecrecover` may return empty address | 1 |
| [QA&#x2011;3](#QA&#x2011;3) | `forceApprove` should be used instead of `approve` | 4 |
| [QA&#x2011;4](#QA&#x2011;4) | The Contract Should `approve(0)` First | 8 |
| [QA&#x2011;5](#QA&#x2011;5) | `address` shouldn't be hard-coded | 1 |
| [QA&#x2011;6](#QA&#x2011;6) | Change contract's `mint` model | 1 |
| [QA&#x2011;7](#QA&#x2011;7) | Functions calling contracts/addresses with transfer hooks are missing reentrancy guards | 2 |
| [QA&#x2011;8](#QA&#x2011;8) | A function which defines named returns in it's declaration doesn't need to use return | 42 |
| [QA&#x2011;9](#QA&#x2011;9) | Use `delete` to Clear Variables | 6 |
| [QA&#x2011;10](#QA&#x2011;10) | Incorrect withdraw declaration | 2 |
| [QA&#x2011;11](#QA&#x2011;11) | Off-by-one timestamp error | 8 |
| [QA&#x2011;12](#QA&#x2011;12) | Redundant Cast | 1 |
| [QA&#x2011;13](#QA&#x2011;13) | Indexed strings should be removed | 1 |

Total: 78 contexts over 13 issues

## QA Issues

### <a href="#qa-summary">[QA&#x2011;1]</a><a name="QA&#x2011;1"> `address wallet` in `USDS.mintTo()` can be `address(collateralAndLiquidity)`

Since the only caller can be `address(collateralAndLiquidity)` according to `require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );`
We can see that in `CollateralAndLiquidity.borrowUSDS()` calls for `usds.mintTo( msg.sender, amountBorrowed );`
As a result, we can remove `address wallet` and simply call to `_mint(address(collateralAndLiquidity), amount);`

#### <ins>Proof Of Concept</ins>

```solidity
File: USDS.sol

40: function mintTo( address wallet, uint256 amount ) external
		{
		require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
		require( amount > 0, "Cannot mint zero USDS" );

		_mint( wallet, amount );

		emit USDSMinted(wallet, amount);
		}

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/USDS.sol#L40



### <a href="#qa-summary">[QA&#x2011;2]</a><a name="QA&#x2011;2"> `ecrecover` may return empty address

There is a common issue that ecrecover returns empty (0x0) address when the signature is invalid. function `_verifySignature` should check that before returning the result of ecrecover.
Should revert when `recoveredAddress` is zero.

#### <ins>Proof Of Concept</ins>


```solidity
File: SigningTools.sol

26: address recoveredAddress = ecrecover(messageHash, v, r, s);
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/SigningTools.sol#L26



### <a href="#qa-summary">[QA&#x2011;3]</a><a name="QA&#x2011;3"> `forceApprove` should be used instead of `approve`

In order to prevent front running, `forceApprove` should be used in place of `approve` where possible 

#### <ins>Proof Of Concept</ins>


```solidity
File: Liquidity.sol

58: tokenA.approve( address(pools), swapAmountA );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L58

```solidity
File: Liquidity.sol

68: tokenB.approve( address(pools), swapAmountB );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L68

```solidity
File: Liquidity.sol

96: tokenA.approve( address(pools), maxAmountA );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L96

```solidity
File: Liquidity.sol

97: tokenB.approve( address(pools), maxAmountB );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L97



### <a href="#qa-summary">[QA&#x2011;4]</a><a name="QA&#x2011;4"> The Contract Should `approve(0)` First

Some tokens (like USDT L199) do not work when changing the allowance from an existing non-zero allowance value. They must first be approved by zero and then the actual allowance must be approved.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: Liquidity.sol

58: tokenA.approve( address(pools), swapAmountA );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L58

```solidity
File: Liquidity.sol

68: tokenB.approve( address(pools), swapAmountB );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L68

```solidity
File: Liquidity.sol

96: tokenA.approve( address(pools), maxAmountA );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L96

```solidity
File: Liquidity.sol

97: tokenB.approve( address(pools), maxAmountB );
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L97



</details>

#### <ins>Recommended Mitigation Steps</ins>

Approve with a zero amount first before setting the actual amount.



## Non Critical Issues

### <a href="#qa-summary">[QA&#x2011;5]</a><a name="QA&#x2011;5"> `address` shouldn't be hard-coded

It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

#### <ins>Proof Of Concept</ins>


```solidity
File: SigningTools.sol

7: address constant public EXPECTED_SIGNER = 0x1234519DCA2ef23207E1CA7fd70b96f281893bAa;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/SigningTools.sol#L7



### <a href="#qa-summary">[QA&#x2011;6]</a><a name="QA&#x2011;6"> Change contract's `mint` model

Instead of having a fixed total supply of the contract's token and minting the entire total supply to a single address, use the minting model at the time of transaction, so total supply will only be as much as the token in use and there will be no tokens minted in a single address.

One advantage of using the constant for total supply in the affected contracts is that it allows you to specify the total supply of tokens in a single place, which can make it easier to maintain and understand the contract.

One disadvantage is that the total supply of tokens is hard-coded into the contract and cannot be changed later. This means that if the total supply needs to be changed for any reason (e.g., to issue more tokens or to burn existing tokens), the contract will need to be modified and redeployed. This can be time-consuming and costly.

Another potential disadvantage is that the use of a constant value for the total supply may make the contract less flexible. For example, if the total supply needs to be determined dynamically based on some external factor (e.g., the total amount of funds raised in a crowdfunding campaign), the contract will need to be modified to accommodate this.

Finally, using a constant value for the total supply may make it more difficult to implement certain features or behaviors that depend on the total supply, such as vesting or token burns. These features may require additional logic to be implemented in the contract, which could increase its complexity and gas cost.

#### <ins>Proof Of Concept</ins>

```solidity
File: Salt.sol

13: uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/Salt.sol#L13



### <a href="#qa-summary">[QA&#x2011;7]</a><a name="QA&#x2011;7"> Functions calling contracts/addresses with transfer hooks are missing reentrancy guards

While adherence to the check-effects-interaction pattern is commendable, the absence of a reentrancy guard in functions, especially where transfer hooks might be present, can expose the protocol users to risks of read-only reentrancies. Such reentrancy vulnerabilities can be exploited to execute malicious actions even without altering the contract state. Without a reentrancy guard, the only potential mitigation would be to blocklist the entire protocol - an extreme and disruptive measure. Therefore, incorporating a reentrancy guard into these functions is vital to bolster security, as it helps protect against both traditional reentrancy attacks and read-only reentrancies, ensuring robust and safe protocol operations.

#### <ins>Proof Of Concept</ins>

```solidity
File: DAO.sol


```solidity
File: DAO.sol

360: function withdrawPOL( IERC20 tokenA, IERC20 tokenB, uint256 percentToLiquidate ) external
		{
		require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );

		bytes32 poolID = PoolUtils._poolID(tokenA, tokenB);
		uint256 liquidityHeld = collateralAndLiquidity.userShareForPool( address(this), poolID );
		if ( liquidityHeld == 0 )
			return;

		uint256 liquidityToWithdraw = (liquidityHeld * percentToLiquidate) / 100;

		
		(uint256 reclaimedA, uint256 reclaimedB) = collateralAndLiquidity.withdrawLiquidityAndClaim(tokenA, tokenB, liquidityToWithdraw, 0, 0, block.timestamp );

		
		tokenA.safeTransfer( address(liquidizer), reclaimedA );
		tokenB.safeTransfer( address(liquidizer), reclaimedB );

		emit POLWithdrawn(tokenA, tokenB, reclaimedA, reclaimedB);
		}

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L360

```solidity
File: Liquidity.sol

121: function _withdrawLiquidityAndClaim( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToWithdraw, uint256 minReclaimedA, uint256 minReclaimedB ) internal returns (uint256 reclaimedA, uint256 reclaimedB)
		{
		bytes32 poolID = PoolUtils._poolID( tokenA, tokenB );
		require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );

		
		
		(reclaimedA, reclaimedB) = pools.removeLiquidity( tokenA, tokenB, liquidityToWithdraw, minReclaimedA, minReclaimedB, totalShares[poolID] );

		
		tokenA.safeTransfer( msg.sender, reclaimedA );
		tokenB.safeTransfer( msg.sender, reclaimedB );

		
		
		
		_decreaseUserShare( msg.sender, poolID, liquidityToWithdraw, true );

		emit LiquidityWithdrawn(msg.sender, address(tokenA), address(tokenB), reclaimedA, reclaimedB, liquidityToWithdraw);
		}

```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L121




### <a href="#qa-summary">[QA&#x2011;8]</a><a name="QA&#x2011;8"> A function which defines named returns in it's declaration doesn't need to use return

Remove the return statement once ensuring it is safe to do so

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArbitrageSearch.sol

37: return (wbtc, salt);

43: return (salt, wbtc);

48: return (wbtc, swapTokenOut);

53: return (swapTokenIn, wbtc);

57: return (swapTokenOut, swapTokenIn);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L37

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L43

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L48

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L53

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L57



```solidity
File: ArbitrageSearch.sol

76: return false;

88: return profitRightOfMidpoint > profitMidpoint;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L76

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L88



```solidity
File: ArbitrageSearch.sol

108: return 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/arbitrage/ArbitrageSearch.sol#L108

```solidity
File: DAO.sol

302: return 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L302

```solidity
File: Proposals.sol

126: return _possiblyCreateProposal( ballotName, ballotType, address1, 0, string1, description );

158: return _possiblyCreateProposal( ballotName, BallotType.PARAMETER, address(0), parameterType, "", description );

176: return ballotID;

190: return _possiblyCreateProposal( ballotName, BallotType.UNWHITELIST_TOKEN, address(token), 0, tokenIconURL, description );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L126

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L158

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L176

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L190



```solidity
File: Proposals.sol

208: return _possiblyCreateProposal( ballotName, BallotType.SEND_SALT, wallet, amount, "", description );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L208

```solidity
File: Proposals.sol

218: return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );

227: return _possiblyCreateProposal( ballotName, BallotType.INCLUDE_COUNTRY, address(0), 0, country, description );

236: return _possiblyCreateProposal( ballotName, BallotType.EXCLUDE_COUNTRY, address(0), 0, country, description );

245: return _possiblyCreateProposal( ballotName, BallotType.SET_CONTRACT, newAddress, 0, "", description );

254: return _possiblyCreateProposal( ballotName, BallotType.SET_WEBSITE_URL, address(0), 0, newWebsiteURL, description );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L218

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L227

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L236

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L245

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L254



```solidity
File: Proposals.sol

299: return ballots[ballotID];

305: return _lastUserVoteForBallot[ballotID][user];

311: return _votesCastForBallot[ballotID][vote];


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L299

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L305

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L311



```solidity
File: Proposals.sol

348: return votes[Vote.INCREASE] + votes[Vote.DECREASE] + votes[Vote.NO_CHANGE];

350: return votes[Vote.YES] + votes[Vote.NO];


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L348

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L350



```solidity
File: PoolMath.sol

173: return 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L173

```solidity
File: PoolMath.sol

224: return (0, 0);

226: return (swapAmountA, swapAmountB);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L224

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolMath.sol#L226



```solidity
File: Pools.sol

104: return ( maxAmount0, maxAmount1, (maxAmount0 + maxAmount1) );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L104

```solidity
File: Pools.sol

305: return swapAmountIn;

311: return 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L305

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L311



```solidity
File: Pools.sol

326: return 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L326

```solidity
File: PoolStats.sol

68: return uint64(i);

71: return INVALID_POOL_ID;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L68

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L71



```solidity
File: PoolStats.sol

145: return _arbitrageIndicies[poolID];


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L145

```solidity
File: PoolUtils.sol

25: return keccak256(abi.encodePacked(address(tokenB), address(tokenA)));

27: return keccak256(abi.encodePacked(address(tokenA), address(tokenB)));


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L25

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L27



```solidity
File: PoolUtils.sol

36: return (keccak256(abi.encodePacked(address(tokenB), address(tokenA))), true);

38: return (keccak256(abi.encodePacked(address(tokenA), address(tokenB))), false);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L36

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L38



```solidity
File: PoolUtils.sol

57: return (0, 0);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolUtils.sol#L57

```solidity
File: Liquidity.sol

75: return (zapAmountA, zapAmountB);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L75

```solidity
File: Liquidity.sol

150: return _depositLiquidityAndIncreaseShare(tokenA, tokenB, maxAmountA, maxAmountB, minLiquidityReceived, useZapping);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L150

```solidity
File: Liquidity.sol

161: return _withdrawLiquidityAndClaim(tokenA, tokenB, liquidityToWithdraw, minReclaimedA, minReclaimedB);


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Liquidity.sol#L161



</details>



### <a href="#qa-summary">[QA&#x2011;9]</a><a name="QA&#x2011;9"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: PoolStats.sol

54: _arbitrageProfits[ poolIDs[i] ] = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/PoolStats.sol#L54

```solidity
File: CoreChainlinkFeed.sol

30: int256 price = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/price_feed/CoreChainlinkFeed.sol#L30

```solidity
File: CollateralAndLiquidity.sol

184: usdsBorrowedByUsers[wallet] = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/CollateralAndLiquidity.sol#L184

```solidity
File: Liquidizer.sol

113: usdsThatShouldBeBurned = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/stable/Liquidizer.sol#L113

```solidity
File: StakingRewards.sol

151: claimableRewards = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L151

```solidity
File: StakingRewards.sol

301: cooldowns[i] = 0;


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L301



</details>




### <a href="#qa-summary">[QA&#x2011;10]</a><a name="QA&#x2011;10"> Incorrect withdraw declaration

In Solidity, it's essential for clarity and interoperability to correctly specify return types in function declarations. If the `withdraw` function is expected to return a `bool` to indicate success or failure, its omission could lead to ambiguity or unexpected behavior when interacting with or calling this function from other contracts or off-chain systems. Missing return values can mislead developers and potentially lead to contract integrations built on incorrect assumptions. To resolve this, the function declaration for `withdraw` should be modified to explicitly include the `bool` return type, ensuring clarity and correctness in contract interactions.

#### <ins>Proof Of Concept</ins>


```solidity
File: DAO.sol

360: function withdrawPOL( IERC20 tokenA, IERC20 tokenB, uint256 percentToLiquidate ) external
		{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L360

```solidity
File: Pools.sol

219: function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L219




### <a href="#qa-summary">[QA&#x2011;11]</a><a name="QA&#x2011;11"> Off-by-one timestamp error

In Solidity, using `>=` or `<=` to compare against `block.timestamp` (alias `now`) may introduce off-by-one errors due to the fact that `block.timestamp` is only updated once per block and its value remains constant throughout the block's execution. If an operation happens at the exact second when `block.timestamp` changes, it could result in unexpected behavior. To avoid this, it's safer to use strict inequality operators (`>` or `<`). For instance, if a condition should only be met after a certain time, use `block.timestamp > time` rather than `block.timestamp >= time`. This way, potential off-by-one errors due to the exact timing of block mining are mitigated, leading to safer, more predictable contract behavior.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ManagedWallet.sol

77: require( block.timestamp >= activeTimelock, "Timelock not yet completed" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/ManagedWallet.sol#L77

```solidity
File: Proposals.sol

83: require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L83

```solidity
File: BootstrapBallot.sol

72: require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/launch/BootstrapBallot.sol#L72

```solidity
File: Pools.sol

83: require(block.timestamp <= deadline, "TX EXPIRED");


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/pools/Pools.sol#L83

```solidity
File: Staking.sol

105: require( block.timestamp >= u.completionTime, "Unstake has not completed yet" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/Staking.sol#L105

```solidity
File: StakingRewards.sol

67: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L67

```solidity
File: StakingRewards.sol

107: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L107

```solidity
File: StakingRewards.sol

300: if ( block.timestamp >= cooldownExpiration )


```

https://github.com/code-423n4/2024-01-salty/tree/main/src/staking/StakingRewards.sol#L300



</details>



### <a href="#qa-summary">[QA&#x2011;12]</a><a name="QA&#x2011;12"> Redundant Cast

The type of the variable is the same as the type to which the variable is being cast

#### <ins>Proof Of Concept</ins>


```solidity
File: Proposals.sol

217: address(contractAddress)
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/Proposals.sol#L217



### <a href="#qa-summary">[QA&#x2011;13]</a><a name="QA&#x2011;13"> Indexed strings

If the protocol consider to parse events from the blockchain, it wouldn't be possible to extract a real (unhashed) value from indexed strings. The filter option could be somehow used only for non string-indexed arguments.

At Solidity keccak256 of indexed string is stored when emitting an event.
(https://docs.soliditylang.org/en/v0.8.19/abi-spec.html#encoding-of-indexed-event-parameters)

#### <ins>Proof Of Concept</ins>

```solidity
File: DAO.sol

26: event SetContract(string indexed ballotName, address indexed contractAddress);
```

https://github.com/code-423n4/2024-01-salty/tree/main/src/dao/DAO.sol#L26


#### <ins>Recommended Mitigation Steps</ins>

Remove `indexed` keyword

