[L-1] The `ManagedWallet` is not functioning as anticipated.

The proposed `mainWallet` can confirm the change only after the `30` days timelock.
However, it can be skipped.

- The `confirmationWallet` is capable of sending `0.05` ether before suggesting a new `main wallet`. 
Subsequently, the `activeTimelock` is updated.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L66
```
receive() external payable {
    if ( msg.value >= .05 ether )
        activeTimelock = block.timestamp + TIMELOCK_DURATION; 
}
```
- After `30` days, the main wallet proposes a new wallet.
- The newly proposed main wallet can confirm the change immediately since `30` days have elapsed.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L77
```
function changeWallets() external {
    require( msg.sender == proposedMainWallet, "Invalid sender" );
    require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
}
```

[L-2] The parameter order in the `proposeCallContract` function is incorrect.

- In the `possiblyCreateProposal` function, the parameter `string2` is associated with the `description`.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L109
```
function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID) {
    ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime );
}
```
- However, in the `proposeCallContract` function, the `description` is inserted as parameter `string1`.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L218
```
function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID) {
    return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
}
```

[L-3] Stop the emission of rewards to the `stakingRewards` when there are no liquidity holdings.

- Upon whitelisting a new token, the `bootstrapping rewards` are directed to the `liquidityRewardsEmitter` for the `WETH/token` and `WBTC/token` pools.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L266
```
function _finalizeTokenWhitelisting( uint256 ballotID ) internal {
    AddedReward[] memory addedRewards = new AddedReward[](2);
    addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
    addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

    exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
    liquidityRewardsEmitter.addSALTRewards( addedRewards );
}
```
- In the `rewards emitter`, rewards are emitted to the pools regardless of whether there are liquidities present in the pools.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L136
```
function performUpkeep( uint256 timeSinceLastUpkeep ) external {
    bytes32[] memory poolIDs;
    if ( isForCollateralAndLiquidity )
        {
        poolIDs = poolsConfig.whitelistedPools();
	}
    else
	{
	poolIDs = new bytes32[](1);
	poolIDs[0] = PoolUtils.STAKED_SALT;
	}
    stakingRewards.addSALTRewards( addedRewards );
}
```
- Consequently, the initial depositor has the potential to receive all rewards emitted.

[L-4] The pools cannot receive rewards when they are unwhitelisted.

- The distribution of `SALT` rewards to the pools is based on arbitrage profits.
- Let's consider a scenario where a specific token becomes unwhitelisted.
In this case, these pools will not be taken into account in the calculation of arbitrage profits in the future.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L71
```
function unwhitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner {
    pools.updateArbitrageIndicies();
}
```
- During upkeep, only the whitelisted pools are taken into consideration. 
Consequently, if unwhitelisted pools generate arbitrage profits, they will not receive the rewards they rightfully deserve.
Additionally, the `WBTC/WETH` pool may also receive smaller rewards than anticipated.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L196-L198
```
function step7() public onlySameContract {
    uint256[] memory profitsForPools = pools.profitsForWhitelistedPools();
    bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
    saltRewards.performUpkeep(poolIDs, profitsForPools );
}
```
Execute `performUpkeep` before unwhitelisting pools.

[L-5] The value of shares may increase.

- The user can deposit `(101, 1e18)` liquidity into the newly created pool.
And, as a result, he will receive `(101 + 1e18)` shares.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L104
```
function _addLiquidity( bytes32 poolID, uint256 maxAmount0, uint256 maxAmount1, uint256 totalLiquidity ) internal returns(uint256 addedAmount0, uint256 addedAmount1, uint256 addedLiquidity) {
    if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
        {
	return ( maxAmount0, maxAmount1, (maxAmount0 + maxAmount1) );
	}
}
```
- If the prices of the two tokens are the same, others can swap tokens in this pool, resulting in reserves of `(1e10, 1e10)`.
- When users deposit a substantial amount, their shares become larger. 
For instance, if a user deposits `(1e18, 1e18)`, the share would be `1e26`. 
It's worth noting that the type of `userShare` is `uint128`, so there is a possibility of overflow in such cases.

[L-6] Swaps in unwhitelisted pools can impact the accurate emission of rewards.

- Let's consider a scenario where the pools become unwhitelisted, and there are existing liquidities in them.
- When swaps occur in those pools, arbitrage profits continue to be generated.
- After performing upkeep, we have only cleared the whitelisted pools.
Consequently, the `_arbitrageProfits` for the unwhitelisted pools become larger.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L51
```
function clearProfitsForPools() external {
    bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
    for( uint256 i = 0; i < poolIDs.length; i++ )
	_arbitrageProfits[ poolIDs[i] ] = 0;
}
```
- Once these pools become whitelisted again, their `_arbitrageProfits` will influence the subsequent rewards emission, and these pools will receive a significant portion of those rewards.

[L-7] There is no access check for `airdroppers`.

- `Airdroppers` can claim initial `SALTs` and stake them immediately, but there is no access check for them.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L81-L82
```
function claimAirdrop() external nonReentrant {
    staking.stakeSALT( saltAmountForEachUser );
    staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );
}
```

[L-8] There is no access check for a user attempting to liquidate another user.

- The absence of an access check allows users without the proper access rights to liquidate other users and receive rewards.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L142-L145
```
function liquidateUser( address wallet ) external nonReentrant {
    require( wallet != msg.sender, "Cannot liquidate self" );
    // First, make sure that the user's collateral ratio is below the required level
    require( canUserBeLiquidated(wallet), "User cannot be liquidated" );
}
```

[L-9] When there is a sufficient balance of `USDS` in `Liquidizer`, there is no need to swap `WBTC` and `WETH` for `USDS`.

- When liquidating a user, we transfer their collateral to the `Liquidizer`.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L176-L177
```
function liquidateUser( address wallet ) external nonReentrant {
    wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );
    weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );
}
```
- Within the Liquidizer, we perform swaps of `WETH` and `WBTC` for `USDS`.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L139-L140
```
function performUpkeep() external {
    PoolUtils._placeInternalSwap(pools, wbtc, usds, wbtc.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );
    PoolUtils._placeInternalSwap(pools, weth, usds, weth.balanceOf(address(this)), maximumInternalSwapPercentTimes1000 );
}
```
In many cases, the price of the collateral is higher than the liquidated `USDS`.
Therefore, the `Liquidizer` might have enough `USDS` to burn. 
It's advisable to retain `WBTC` and `WETH` in the `Liquidize`r and execute swaps only when necessary.

[L-10] Certain steps in the `performUpkeep` function are interrelated.

- In the `performUpkeep` function, we treat all steps as independent.
 Even if some steps are reverted, the overall transaction will still proceed.
- But some steps are related. 
We can use steps `3`, `4`, and `5` as an example.
In step `4`, we send `20%` of the current `WETH` for `SALT/USDS` `POL` and convert the remaining `WETH` to `SALT`. 
However, if step `4` is reverted, the entire `WETH` would be converted to `SALT` rewards.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L158-L180
```
function step4() public onlySameContract {
    uint256 wethBalance = weth.balanceOf( address(this) );
    uint256 amountOfWETH = wethBalance * daoConfig.arbitrageProfitsPercentPOL() / 100;
    _formPOL(salt, usds, amountOfWETH);
}
function step5() public onlySameContract {
    uint256 wethBalance = weth.balanceOf( address(this) );
    uint256 amountSALT = pools.depositSwapWithdraw( weth, salt, wethBalance, 0, block.timestamp );
    salt.safeTransfer(address(saltRewards), amountSALT);
}
```

[L-11] There is currently no method to withdraw tokens from the `DAO`'s `POL`.

- The `SALT/USDS` and `USDS/DAI` `POL`s are consistently established using arbitrage profits.
- These pools are utilized to augment the `USDS` buffer for the `Liquidizer`.
- However, if there are substantial liquidities in these `POL`s, it might be beneficial to withdraw a portion of them.

[L-12] It would be more advantageous to allocate `SALT`s as rewards in the `Liquidizer`.
- In the `Liquidizer`, if there is an insufficient amount of `USDS` for burning, it withdraws `POL`s and converts them to `USDS`. 
However, the `SALT`s are burned in this process.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L144-L149
```
function performUpkeep() external {
    uint256 saltBalance = salt.balanceOf(address(this));
    if ( saltBalance > 0 )
        {
	salt.safeTransfer(address(salt), saltBalance);
	salt.burnTokensInContract();
	}
}
```
- Considering that these `POL`s are established using arbitrage profits, it would be more advantageous to send these `SALT`s to the `rewardsEmitter` rather than burning them.

[L-13] There is no access check for users who submit a proposal.

- If a user has staked some `SALT`s and becomes inaccessible, they can still make a proposal. 
It's unclear if this behavior is intentional.

[L-14] There is an unnecessary call to the `excludedCountriesUpdated` function.
- When a country is excluded, we update the geo version, necessitating new signatures for all users.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L197
```
function _executeApproval( Ballot memory ballot ) internal {
    excludedCountries[ ballot.string1 ] = true;			 
    exchangeConfig.accessManager().excludedCountriesUpdated();
}
```
- But if a country that is already excluded is excluded again, this function call becomes unnecessary and adds unnecessary complexity to the protocol.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/AccessManager.sol#L41C4-L46C7
```
function excludedCountriesUpdated() external {
    require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
    geoVersion += 1;
}
```

[L-15] Pools have the potential to receive bootstrapping rewards multiple times.

- Upon whitelisting new pools, bootstrapping rewards are allocated to them.
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L265-L266
```
function _finalizeTokenWhitelisting( uint256 ballotID ) internal {
    AddedReward[] memory addedRewards = new AddedReward[](2);
    addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
    addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

    exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
    liquidityRewardsEmitter.addSALTRewards( addedRewards );
}
```
- If those pools become unwhitelisted and are later whitelisted again, new bootstrapping rewards are allocated to them.
This implies that `LPs` for these pools can receive more bootstrapping rewards. 
To address this, we should implement a restriction to ensure that any pool receives bootstrapping rewards only once.