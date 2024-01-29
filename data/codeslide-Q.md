## <a id="summary"></a>Summary

### <a id="summary-low-risk-issues"></a>Low Risk Issues
There are 73 instances over 8 issues.
|      ID       | Description                                                                             | Count |
| :-----------: | :-------------------------------------------------------------------------------------- | ----: |
| [L-01](#l-01) | Setters should have input validation                                                    |     5 |
| [L-02](#l-02) | Missing checks for `address(0x0)` when updating `address` state variables               |     3 |
| [L-03](#l-03) | Some tokens may revert when zero value transfers are made                               |     9 |
| [L-04](#l-04) | External calls in an un-bounded `for`-loop may result in a DOS                          |     4 |
| [L-05](#l-05) | Contracts use infinite approvals with no means to revoke                                |     9 |
| [L-06](#l-06) | Functions calling contracts/addresses with transfer hooks are missing reentrancy guards |     9 |
| [L-07](#l-07) | Large approvals may not work with some `ERC20` tokens                                   |    12 |
| [L-08](#l-08) | Loss of precision                                                                       |    22 |

### <a id="summary-non-critical-issues"></a>Non-Critical Issues
There are 17 instances over 3 issues.
|       ID        | Description                                                                | Count |
| :-------------: | :------------------------------------------------------------------------- | ----: |
| [NC-01](#nc-01) | Missing event for critical parameter change                                |     1 |
| [NC-02](#nc-02) | Consider bounding input array length                                       |     2 |
| [NC-03](#nc-03) | Not using the named return variables anywhere in the function is confusing |    14 |


## <a id="details"></a>Details

### <a id="details-low-risk-issues"></a>Low Risk Issues

#### <a id="l-01"></a>\[L-01\] Setters should have input validation
##### Description:
When setting the value of a state variable, validation should be included to prevent setting an invalid value. Even when there is access control on the setter function, a mistake/typo can allow the setting of an invalid value and lead to an undesired state for the contract. Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Add validation to prevent setting an invalid value.

##### Instances:
There are 5 instances of this issue.
<details><summary>View 5 Instances</summary>

```solidity
File: src/ExchangeConfig.sol

/// @audit state variables: dao, upkeep, initialDistribution, airdrop, teamVestingWallet, daoVestingWallet
48: 	function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
49: 		{
```

| File Link                                                                                          | Instance Count | Instance Link                                                                          |
| :------------------------------------------------------------------------------------------------- | -------------: | :------------------------------------------------------------------------------------- |
| [ExchangeConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol) |              1 | [48](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L48) |
___
```solidity
File: src/pools/Pools.sol

/// @audit state variables: dao, collateralAndLiquidity
60: 	function setContracts( IDAO _dao, ICollateralAndLiquidity _collateralAndLiquidity ) external onlyOwner
61: 		{
```

| File Link                                                                              | Instance Count | Instance Link                                                                       |
| :------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------- |
| [Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol) |              1 | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L60) |
___
```solidity
File: src/price_feed/PriceAggregator.sol

/// @audit state variables: priceFeed1, priceFeed2, priceFeed3
37: 	function setInitialFeeds( IPriceFeed _priceFeed1, IPriceFeed _priceFeed2, IPriceFeed _priceFeed3 ) public onlyOwner
38: 		{

/// @audit state variable: priceFeed1
47: 	function setPriceFeed( uint256 priceFeedNum, IPriceFeed newPriceFeed ) public onlyOwner
48: 		{
```

| File Link                                                                                                       | Instance Count | Instance Links                                                                                                                                                                                        |
| :-------------------------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol) |              2 | [37](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L37),[47](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L47) |
___
```solidity
File: src/stable/Liquidizer.sol

/// @audit state variables: collateralAndLiquidity, pools, dao
63: 	function setContracts( ICollateralAndLiquidity _collateralAndLiquidity, IPools _pools, IDAO _dao) external onlyOwner
64: 		{
```

| File Link                                                                                         | Instance Count | Instance Link                                                                             |
| :------------------------------------------------------------------------------------------------ | -------------: | :---------------------------------------------------------------------------------------- |
| [Liquidizer.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol) |              1 | [63](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L63) |
___
</details>

#### <a id="l-02"></a> \[L-02\] Missing checks for `address(0x0)` when updating `address` state variables
##### Description:
Lack of zero-address validation on `address` parameters may lead to transaction reverts, wastes gas, may require resubmission of transactions, and may force contract redeployments in certain cases within the protocol. Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Consider adding explicit zero-address validation prior to assignment of a value to an `address` state variable.

##### Instances:
There are 3 instances of this issue.
<details><summary>View 3 Instances</summary>

```solidity
File: src/price_feed/PriceAggregator.sol

/// @audit function setPriceFeed()
54: 			priceFeed1 = newPriceFeed;

/// @audit function setPriceFeed()
56: 			priceFeed2 = newPriceFeed;

/// @audit function setPriceFeed()
58: 			priceFeed3 = newPriceFeed;
```

| File Link                                                                                                       | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| :-------------------------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol) |              6 | [41](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L41),[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L42),[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L43),[54](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L54),[56](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L56),[58](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L58) |
___
</details>

#### <a id="l-03"></a> \[L-03\] Some tokens may revert when zero value transfers are made
##### Description:
In spite of the fact that [EIP-20 states](https://github.com/ethereum/EIPs/blob/46b9b698815abbfa628cd1097311deee77dd45c5/EIPS/eip-20.md?plain=1#L116) that zero-valued transfers must be accepted, some tokens, such as [LEND](https://etherscan.io/token/0x80fB784B7eD66730e8b1DBd9820aFD29931aab03) will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. See [Weird ERC20 Tokens - Revert on Zero Value Transfers](https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers). Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Consider skipping the transfer if the amount is zero, which will also save gas.

##### Instances:
There are 9 instances of this issue.
<details><summary>View 9 Instances</summary>

```solidity
File: src/Upkeep.sol

179: 		salt.safeTransfer(address(saltRewards), amountSALT);

238: 		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
```

| File Link                                                                          | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                      |
| :--------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol) |              4 | [136](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L136),[137](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L137),[179](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L179),[238](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L238) |
___
```solidity
File: src/dao/DAO.sol

342: 		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), amountToSendToTeam );

350: 		salt.safeTransfer( address(salt), saltToBurn );

375: 		tokenA.safeTransfer( address(liquidizer), reclaimedA );

376: 		tokenB.safeTransfer( address(liquidizer), reclaimedB );
```

| File Link                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                          |
| :------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) |              4 | [342](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L342),[350](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L350),[375](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L375),[376](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L376) |
___
```solidity
File: src/stable/CollateralAndLiquidity.sol

172: 		wbtc.safeTransfer( msg.sender, rewardedWBTC );
```

| File Link                                                                                                                 | Instance Count | Instance Link                                                                                           |
| :------------------------------------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------------------------ |
| [CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol) |              1 | [172](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L172) |
___
```solidity
File: src/stable/Liquidizer.sol

94: 		usds.safeTransfer( address(usds), amountToBurn );
```

| File Link                                                                                         | Instance Count | Instance Link                                                                             |
| :------------------------------------------------------------------------------------------------ | -------------: | :---------------------------------------------------------------------------------------- |
| [Liquidizer.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol) |              1 | [94](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L94) |
___
```solidity
File: src/staking/Staking.sol

50: 		salt.safeTransferFrom( msg.sender, address(this), amountToStake );
```

| File Link                                                                                    | Instance Count | Instance Link                                                                           |
| :------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------- |
| [Staking.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol) |              1 | [50](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L50) |
___
</details>

#### <a id="l-04"></a> \[L-04\] External calls in an un-bounded `for`-loop may result in a DOS
##### Description:
Using external calls in an unbounded `for` loop may result in a denial of service (DOS). Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Consider limiting the number of iterations in `for` loops that make external calls.

##### Instances:
There are 4 instances of this issue.
```solidity
File: src/pools/PoolStats.sol

/// @audit underlyingTokenPair() on line 84
81: 		for( uint256 i = 0; i < poolIDs.length; i++ )
82: 			{
```

| File Link                                                                                      | Instance Count | Instance Link                                                                           |
| :--------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------- |
| [PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol) |              1 | [81](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81) |
___
```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit isWhitelisted() on line 63
60: 		for( uint256 i = 0; i < addedRewards.length; i++ )
61: 			{
```

| File Link                                                                                                  | Instance Count | Instance Link                                                                                  |
| :--------------------------------------------------------------------------------------------------------- | -------------: | :--------------------------------------------------------------------------------------------- |
| [RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol) |              1 | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L60) |
___
```solidity
File: src/stable/CollateralAndLiquidity.sol

/// @audit minimumCollateralRatioPercent() on line 326
321: 			for ( uint256 i = startIndex; i <= endIndex; i++ )
322: 				{
```

| File Link                                                                                                                 | Instance Count | Instance Link                                                                                           |
| :------------------------------------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------------------------ |
| [CollateralAndLiquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol) |              1 | [321](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321) |
___
```solidity
File: src/staking/StakingRewards.sol

/// @audit isWhitelisted() on line 190
185: 		for( uint256 i = 0; i < addedRewards.length; i++ )
186: 			{
```

| File Link                                                                                                  | Instance Count | Instance Link                                                                                    |
| :--------------------------------------------------------------------------------------------------------- | -------------: | :----------------------------------------------------------------------------------------------- |
| [StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol) |              1 | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185) |
___

#### <a id="l-05"></a> \[L-05\] Contracts use infinite approvals with no means to revoke
##### Description:
Infinite approvals on external contracts [can be dangerous](https://revoke.cash/exploits) if the target becomes compromised. The following contracts are vulnerable to such attacks since they have no functionality to revoke the approval (call `approve` with amount 0). Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Consider adding the ability to revoke approval in emergency situations.

##### Instances:
There are 9 instances of this issue.
<details><summary>View 9 Instances</summary>

```solidity
File: src/dao/DAO.sol

90: 		salt.approve(address(collateralAndLiquidity), type(uint256).max);

91: 		usds.approve(address(collateralAndLiquidity), type(uint256).max);

92: 		dai.approve(address(collateralAndLiquidity), type(uint256).max);

265: 			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
```

| File Link                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                    |
| :------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) |              4 | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L90),[91](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L91),[92](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L92),[265](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L265) |
___
```solidity
File: src/launch/Airdrop.sol

67: 		salt.approve( address(staking), saltBalance );
```

| File Link                                                                                   | Instance Count | Instance Link                                                                          |
| :------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------- |
| [Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol) |              1 | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L67) |
___
```solidity
File: src/staking/Liquidity.sol

58: 			tokenA.approve( address(pools), swapAmountA );

68: 			tokenB.approve( address(pools), swapAmountB );

96: 		tokenA.approve( address(pools), maxAmountA );

97: 		tokenB.approve( address(pools), maxAmountB );
```

| File Link                                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                          |
| :----------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Liquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol) |              4 | [58](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L58),[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L68),[96](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L96),[97](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L97) |
___
</details>

#### <a id="l-06"></a> \[L-06\] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards
##### Description:
Even if the function follows the best practice of the [check-effects-interactions pattern](https://docs.soliditylang.org/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern), not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it other than by block-listing the whole protocol.

##### Recommendation:
Add a reentrancy guard to any function using a transfer hook.

##### Instances:
There are 9 instances of this issue.
<details><summary>View 9 Instances</summary>

```solidity
File: src/Upkeep.sol

/// @audit function: `step2()`
122: 		weth.safeTransfer(receiver, rewardAmount);

/// @audit function: `step5()`
179: 		salt.safeTransfer(address(saltRewards), amountSALT);

/// @audit function: `step11()`
238: 		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
```

| File Link                                                                          | Instance Count | Instance Links                                                                                                                                                                                                                                     |
| :--------------------------------------------------------------------------------- | -------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol) |              3 | [122](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L122),[179](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L179),[238](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L238) |
___
```solidity
File: src/dao/DAO.sol

/// @audit function: `withdrawArbitrageProfits()`
308: 		weth.safeTransfer( msg.sender, withdrawnAmount );

/// @audit function: `processRewardsFromPOL()`
342: 		salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), amountToSendToTeam );

/// @audit function: `withdrawPOL()`
375: 		tokenA.safeTransfer( address(liquidizer), reclaimedA );
```

| File Link                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                        |
| :------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) |              3 | [308](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L308),[342](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L342),[375](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L375) |
___
```solidity
File: src/launch/InitialDistribution.sol

/// @audit function: `distributionApproved()`
56: 		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );
```

| File Link                                                                                                           | Instance Count | Instance Link                                                                                      |
| :------------------------------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------------------- |
| [InitialDistribution.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol) |              1 | [56](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L56) |
___
```solidity
File: src/rewards/Emissions.sol

/// @audit function: `performUpkeep()`
60: 		salt.safeTransfer(address(saltRewards), saltToSend);
```

| File Link                                                                                        | Instance Count | Instance Link                                                                             |
| :----------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------- |
| [Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol) |              1 | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L60) |
___
```solidity
File: src/stable/Liquidizer.sol

/// @audit function: `performUpkeep()`
147: 			salt.safeTransfer(address(salt), saltBalance);
```

| File Link                                                                                         | Instance Count | Instance Link                                                                               |
| :------------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------------ |
| [Liquidizer.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol) |              1 | [147](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L147) |
___
</details>

#### <a id="l-07"></a> \[L-07\] Large approvals may not work with some `ERC20` tokens
##### Description:
Not all `IERC20` implementations are totally compliant, and some (e.g., `UNI`, `COMP`) may fail if the valued passed to `approve` is larger than `uint96`. If the approval amount is `type(uint256).max`, this may cause issues with systems that expect the value passed to approve to be reflected in the allowances mapping. Note: The automated report has this finding, but did not cover these instances.

##### Instances:
There are 12 instances of this issue.
<details><summary>View 12 Instances</summary>

```solidity
File: src/Upkeep.sol

91: 		weth.approve( address(pools), type(uint256).max );
```

| File Link                                                                          | Instance Count | Instance Link                                                                  |
| :--------------------------------------------------------------------------------- | -------------: | :----------------------------------------------------------------------------- |
| [Upkeep.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol) |              1 | [91](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L91) |
___
```solidity
File: src/dao/DAO.sol

90: 		salt.approve(address(collateralAndLiquidity), type(uint256).max);

91: 		usds.approve(address(collateralAndLiquidity), type(uint256).max);

92: 		dai.approve(address(collateralAndLiquidity), type(uint256).max);

/// @audit uint256
265: 			exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
```

| File Link                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                    |
| :------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol) |              4 | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L90),[91](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L91),[92](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L92),[265](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L265) |
___
```solidity
File: src/launch/Airdrop.sol

/// @audit uint256 saltBalance
67: 		salt.approve( address(staking), saltBalance );
```

| File Link                                                                                   | Instance Count | Instance Link                                                                          |
| :------------------------------------------------------------------------------------------ | -------------: | :------------------------------------------------------------------------------------- |
| [Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol) |              1 | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L67) |
___
```solidity
File: src/rewards/RewardsEmitter.sol

50: 		salt.approve(address(stakingRewards), type(uint256).max);
```

| File Link                                                                                                  | Instance Count | Instance Link                                                                                  |
| :--------------------------------------------------------------------------------------------------------- | -------------: | :--------------------------------------------------------------------------------------------- |
| [RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol) |              1 | [50](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L50) |
___
```solidity
File: src/rewards/SaltRewards.sol

41: 		salt.approve( address(stakingRewardsEmitter), type(uint256).max );

42: 		salt.approve( address(liquidityRewardsEmitter), type(uint256).max );
```

| File Link                                                                                            | Instance Count | Instance Links                                                                                                                                                                          |
| :--------------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [SaltRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol) |              2 | [41](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L41),[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L42) |
___
```solidity
File: src/stable/Liquidizer.sol

71: 		wbtc.approve( address(pools), type(uint256).max );

72: 		weth.approve( address(pools), type(uint256).max );

73: 		dai.approve( address(pools), type(uint256).max );
```

| File Link                                                                                         | Instance Count | Instance Links                                                                                                                                                                                                                                                                |
| :------------------------------------------------------------------------------------------------ | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Liquidizer.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol) |              3 | [71](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L71),[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L72),[73](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L73) |
___
</details>

#### <a id="l-08"></a> \[L-08\] Loss of precision
##### Description:
Division by large numbers may result in the quotient being zero, due to Solidity not supporting fractions. Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator.

##### Instances:
There are 22 instances of this issue.
<details><summary>View 22 Instances</summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

68: 			uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);

69: 			amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);

70: 			amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);

82: 			amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);

83: 			amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);

84: 			amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);

129: 			uint256 amountOut = (reservesA1 * bestArbAmountIn) / (reservesA0 + bestArbAmountIn);

130: 			amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);

131: 			amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
```

| File Link                                                                                                      | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| :------------------------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol) |              9 | [68](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L68),[69](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L69),[70](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L70),[82](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L82),[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L83),[84](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L84),[129](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L129),[130](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L130),[131](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L131) |
___
```solidity
File: src/dao/Proposals.sol

90: 			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

324: 			requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

326: 			requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

329: 			requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
```

| File Link                                                                                    | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                |
| :------------------------------------------------------------------------------------------- | -------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol) |              4 | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L90),[324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324),[326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326),[329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329) |
___
```solidity
File: src/pools/PoolMath.sol

192:         uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );

203: 		swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );
```

| File Link                                                                                    | Instance Count | Instance Links                                                                                                                                                                    |
| :------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [PoolMath.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol) |              2 | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192),[203](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L203) |
___
```solidity
File: src/pools/PoolUtils.sol

61: 		uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
```

| File Link                                                                                      | Instance Count | Instance Link                                                                           |
| :--------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------- |
| [PoolUtils.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol) |              1 | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L61) |
___
```solidity
File: src/price_feed/CoreUniswapFeed.sol

70: 			return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );

107: 			uniswapWBTC_WETH = 10**36 / uniswapWBTC_WETH;

110: 			uniswapWETH_USDC = 10**36 / uniswapWETH_USDC;

112: 		return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;
```

| File Link                                                                                                       | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| :-------------------------------------------------------------------------------------------------------------- | -------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [CoreUniswapFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol) |              7 | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59),[70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70),[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L72),[107](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L107),[110](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L110),[112](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112),[126](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L126) |
___
```solidity
File: src/rewards/Emissions.sol

55: 		uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
```

| File Link                                                                                        | Instance Count | Instance Link                                                                             |
| :----------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------- |
| [Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol) |              1 | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55) |
___
```solidity
File: src/staking/Staking.sol

211:     	return numerator / ( 100 * unstakeRange );
```

| File Link                                                                                    | Instance Count | Instance Link                                                                             |
| :------------------------------------------------------------------------------------------- | -------------: | :---------------------------------------------------------------------------------------- |
| [Staking.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol) |              1 | [211](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L211) |
___
</details>









### <a id="details-non-critical-issues"></a> Non-Critical Issues

#### <a id="nc-01"></a>\[NC-01\] Missing event for critical parameter change

##### Description:
Events help off-chain tools to track changes. Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Emit an event for critical parameter changes, including the old and new values.

##### Instances:
There is 1 instance of this issue.

```solidity
File: src/pools/PoolStats.sol

/// @audit state variable changed: _arbitrageIndicies
77: 	function updateArbitrageIndicies() public
78: 		{
```

| File Link                                                                                      | Instance Count | Instance Link                                                                           |
| :--------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------- |
| [PoolStats.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol) |              1 | [77](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L77) |

#### <a id="nc-02"></a> \[NC-02\] Consider bounding input array length
##### Description:
The functions below take in an unbounded array, and make function calls for entries in the array. While the function will revert if it eventually runs out of gas, it may be a better user experience to `require()` that the length of the array is below some reasonable maximum, so that the user does not have to use up a full transaction's gas only to see that the transaction reverts. Note: The automated report has this finding, but did not cover these instances.

##### Instances:
There are 2 instances of this issue.
```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit function: addSALTRewards(), array parameter: addedRewards[]
60: 		for( uint256 i = 0; i < addedRewards.length; i++ )
61: 			{
62: 			AddedReward memory addedReward = addedRewards[i];
63: 			require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );
64:
65: 			uint256 amountToAdd = addedReward.amountToAdd;
66: 			if ( amountToAdd != 0 )
67: 				{
68: 				// Update pendingRewards so the SALT can be distributed later
69: 				pendingRewards[ addedReward.poolID ] += amountToAdd;
70: 				sum += amountToAdd;
71: 				}
72: 			}
```

| File Link                                                                                                  | Instance Count | Instance Link                                                                                  |
| :--------------------------------------------------------------------------------------------------------- | -------------: | :--------------------------------------------------------------------------------------------- |
| [RewardsEmitter.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol) |              1 | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L60) |
___
```solidity
File: src/staking/StakingRewards.sol

/// @audit function: addSALTRewards(), array parameter: addedRewards[]
185: 		for( uint256 i = 0; i < addedRewards.length; i++ )
186: 			{
187: 			AddedReward memory addedReward = addedRewards[i];
188:
189: 			bytes32 poolID = addedReward.poolID;
190: 			require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
191:
192: 			uint256 amountToAdd = addedReward.amountToAdd;
193:
194: 			totalRewards[ poolID ] += amountToAdd;
195: 			sum = sum + amountToAdd;
196:
197: 			emit SaltRewardsAdded(poolID, amountToAdd);
198: 			}
```

| File Link                                                                                                  | Instance Count | Instance Link                                                                                    |
| :--------------------------------------------------------------------------------------------------------- | -------------: | :----------------------------------------------------------------------------------------------- |
| [StakingRewards.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol) |              1 | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185) |
___

#### <a id="nc-03"></a> \[NC-03\] Not using the named return variables anywhere in the function is confusing
##### Description:
When specifying a named return variable, the variable should be used within the function. Note: The automated report has this finding, but did not cover these instances.

##### Recommendation:
Since the return variable is never assigned, nor is it returned by name, consider changing the return value to be unnamed. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.

##### Instances:
There are 14 instances of this issue.
<details><summary>View 14 Instances</summary>

```solidity
File: src/dao/Proposals.sol

/// @audit unused return variable: ballotID
122: 	function createConfirmationProposal( string calldata ballotName, BallotType ballotType, address address1, string calldata string1, string calldata description ) external returns (uint256 ballotID)
123: 		{

/// @audit unused return variable: ballotID
155: 	function proposeParameterBallot( uint256 parameterType, string calldata description ) external nonReentrant returns (uint256 ballotID)
156: 		{

/// @audit unused return variable: _ballotID
162: 	function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID)
163: 		{

/// @audit unused return variable: ballotID
180: 	function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181: 		{

/// @audit unused return variable: ballotID
196: 	function proposeSendSALT( address wallet, uint256 amount, string calldata description ) external nonReentrant returns (uint256 ballotID)
197: 		{

/// @audit unused return variable: ballotID
213: 	function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID)
214: 		{

/// @audit unused return variable: ballotID
222: 	function proposeCountryInclusion( string calldata country, string calldata description ) external nonReentrant returns (uint256 ballotID)
223: 		{

/// @audit unused return variable: ballotID
231: 	function proposeCountryExclusion( string calldata country, string calldata description ) external nonReentrant returns (uint256 ballotID)
232: 		{

/// @audit unused return variable: ballotID
240: 	function proposeSetContractAddress( string calldata contractName, address newAddress, string calldata description ) external nonReentrant returns (uint256 ballotID)
241: 		{

/// @audit unused return variable: ballotID
249: 	function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
250: 		{
```

| File Link                                                                                    | Instance Count | Instance Links                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol) |             10 | [122](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L122),[155](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L155),[162](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L162),[180](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L180),[196](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L196),[213](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L213),[222](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L222),[231](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L231),[240](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L240),[249](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L249) |
___
```solidity
File: src/pools/PoolsConfig.sol

/// @audit unused return variables: tokenA, tokenB
133: 	function underlyingTokenPair( bytes32 poolID ) external view returns (IERC20 tokenA, IERC20 tokenB)
134: 		{
```

| File Link                                                                                          | Instance Count | Instance Link                                                                               |
| :------------------------------------------------------------------------------------------------- | -------------: | :------------------------------------------------------------------------------------------ |
| [PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol) |              1 | [133](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L133) |
___
```solidity
File: src/staking/Liquidity.sol

/// @audit unused return variables: amountForLiquidityA, amountForLiquidityB
50: 	function _dualZapInLiquidity(IERC20 tokenA, IERC20 tokenB, uint256 zapAmountA, uint256 zapAmountB ) internal returns (uint256 amountForLiquidityA, uint256 amountForLiquidityB  )
51: 		{

/// @audit unused return variables: addedAmountA, addedAmountB, addedLiquidity
146: 	function depositLiquidityAndIncreaseShare( IERC20 tokenA, IERC20 tokenB, uint256 maxAmountA, uint256 maxAmountB, uint256 minLiquidityReceived, uint256 deadline, bool useZapping ) external nonReentrant ensureNotExpired(deadline) returns (uint256 addedAmountA, uint256 addedAmountB, uint256 addedLiquidity)
147: 		{

/// @audit unused return variables: reclaimedA, reclaimedB
157:     function withdrawLiquidityAndClaim( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToWithdraw, uint256 minReclaimedA, uint256 minReclaimedB, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 reclaimedA, uint256 reclaimedB)
158:     	{
```

| File Link                                                                                        | Instance Count | Instance Links                                                                                                                                                                                                                                                                    |
| :----------------------------------------------------------------------------------------------- | -------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Liquidity.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol) |              3 | [50](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L50),[146](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L146),[157](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L157) |
___
</details>
