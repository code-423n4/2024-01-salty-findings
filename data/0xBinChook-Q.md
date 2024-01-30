# QA Report for Salty.io

| Id              | Issue                                                                              |
|-----------------|------------------------------------------------------------------------------------|
| [L-1](#l-1)     | Staking configuration changes only applies to new unstake after finalization       |
| [L-2](#l-2)     | hitelisting, un-whitelisting then re-whitelisting to game bootstrap rewards        |
| [L-3](#l-3)     | Price feed proposal are not check to ensure feed uniqueness                        |
| [L-4](#l-4)     | Rounding down prevents proposals when there is between 1 and 999 xSALT             |
| [L-5](#l-5)     | When xSalt is low, there is a risk of proposal spam                                |
| [L-6](#l-6)     | `SigningTools::_verifySignature` allows malleable (non-unique) signature           |
| [L-7](#l-7)     | Salt from unclaimed airdrops will be trapped                                       |
| [L-8](#l-8)     | `ManagedWallet` is an trap ETH                                                     |
| [L-9](#l-9)     | Control of `proposedConfirmationWallet` should be confirmed                        |
| [L-10](#l-10)   | Both proposed wallets can be the same                                              |
| [L-11](#l-11)   | Upkeep has a race condition that creates an economic disincentive                  |
| [NC-1](#nc-1)   | When a user recasts their vote emit an event                                       |
| [NC-2](#nc-2)   | `Proposals::userHasActiveProposal()` will be incorrect for the DAO                 |
| [NC-3](#nc-3)   | `Proposals::openBallots()` returns an unbounded set                                |
| [NC-4](#nc-4)   | Inconsistent quorum tiers between `minimumQuorum` and `requiredQuorum`             |
| [NC-5](#nc-5)   | Integer rounding may leave dust in `Airdrop`                                       |
| [NC-6](#nc-6)   | Hardcoded ETH amount for confirmation signal is brittle                            |
| [NC-7](#nc-7)   | Governance update ro geo-restriction depends on off-chain private party            |
| [NC-8](#nc-8)   | `Staking::unstakesForUser()` enumerates over an unbounded set                      |
| [NC-9](#nc-9)   | `CollateralAndLiquidty::findLiquidatableUsers()` enumerates over an unbounded set  |
| [NC-10](#nc-10) | Exchange can be started with only a single yes vote                                |



## Low Risk 

### L-1
#### Staking configuration changes only applies to new unstake after finalization
to the `StakingConfig` alter the conditions for users wishing to unstake. 
Users who are already unstaking at the time of these changes will not be affected by them if the changes are advantageous, such as a decrease in the time required to unstake their complete share.

##### Recommended Mitigation
Include the `maxUnstakeWeeks` in the `Unstake` struct and provide a function for users to call. to reduce their staking time when the criteria have been altered with governance.



### L-2
#### Whitelisting, un-whitelisting then re-whitelisting to game bootstrap rewards
In the DAO governance process, when a token is approved through a proposal to be whitelisted, it becomes eligible for the `bootstrappingRewards` in SALT. 
There are no safeguards on the `bootstrappingRewards` for the same token being whitelisted again after it has been un-whitelisted, 
allowing the exploited of repeatedly listing and un-whitelisting tokens in order to artificially inflate the rewards received by the pool.

##### Recommended Mitigation
Keep a mapping of un-whitelisted tokens, and only provide `bootstrappingRewards` to no in that mapping.



### L-3
#### Price feed proposal are not check to ensure feed uniqueness
There is no verification to confirm that when the price feed from a proposal is applied, the three price feed addresses are distinct. 
If two or more of these feeds were directed to the same address, it would undermine the purpose of having a redundant feed system. As the two identical feeds would produce agreeing results, eliminating the benefit of having multiple sources for price information.

##### Recommended Mitigation
In `DAO::_executeSetContract()`, before applying the mutation check the three price feeds would remain unique.



### L-4
#### Rounding down prevents proposals when there is between 1 and 999 xSALT
With the lowest required proposal stake configuration, if the sum of staked SALT is between 1 and 999 xSALT then no proposals can be made, even though the proposer may have xSALT.

In [Proposals::_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L90-L92) 
```solidity
    uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

    require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
```
`requiredXSalt` is zero, due to the denominator being larger than the numerator.

##### Recommended Mitigation
If expected behaviour then document in NatSpec, otherwise use a default minimum value for `requiredXSalt` in a similar approach to required quorum. 



### L-5
#### When xSalt is low, there is a risk of proposal spam
The threshold for `requiredXSalt` is calculated using the current amount of xSalt (staked amount of SALT).

In [Proposals::_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L89-L91)
```solidity
    uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
    uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
```

When xSalt is low, the `requiredXSalt` will be even lower (`daoConfig.requiredProposalPercentStakeTimes1000()` ranges from 0.1% to 2%), with a lower bar cost to create a spam proposal (or self-interested one) more will get created.

##### Recommended Mitigation
Implement a default minimum value for `requiredXSalt` in a similar approach to required quorum, where if `requiredXSalt` is below the minimum use that instead.



### L-6
#### `SigningTools::_verifySignature` allows malleable (non-unique) signature
The signature verification used during the `bootstrapBallot` and by `AccessManager::grantAccess()` allows malleable (non-unique) signatures, as defined by EIP-2.

##### Recommended Mitigation
Use Open Zeppelin's [ECDSA::tryRecover()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/692dbc560f48b2a5160e6e4f78302bb93314cd88/contracts/utils/cryptography/ECDSA.sol#L128C57-L128C69) or replicate the enforcement of signatures being in the lower order.



### L-7
#### Salt from unclaimed airdrops will be trapped
The first on-chain step for the Salt airdrop, the users must cast their vote with `BootstrapBallot::vote()`, which registers them with `Airdrop::authorizeWallet()`. 
Following a successful bootstrap ballot that initiates the exchange, the second step has users calling `Airdrop::claimAirdrop()`. Failing to do so results in their allocated Salt remaining inaccessible within `Airdrop`.

The trapped Salt will be out of circulation, but still counted as part of the total, artificially inflating the total circulating supply.

##### Recommended Mitigation
Have `Airdop` implement `ICalledContract` with the `callFromDAO` function transferring the outstanding Salt to be burnt, effectively closing the claim window using a DAO Proposal.



### L-8
#### `ManagedWallet` is an trap ETH
Receiving ETH is used as a decision signal by the `confirmationWallet` whenever a wallet change is proposed by the `mainWallet`.

[ManagedWallet::receive()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L63C1-L68C60)
```solidity
    // Confirm if .05 or more ether is sent and otherwise reject.
    // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    if ( msg.value >= .05 ether )
        activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    else
        activeTimelock = type(uint256).max; // effectively never
```
Affirmation requires `0.05 ether` or greater to be transferred to `ManagedWallet`, but there is no mechanism for withdrawing the accused ETH, trapping it in the `ManagedWallet`.

##### Recommended Mitigation
Add a `withdraw()` that allows either the `mainWallet` or `confirmationWallet` to pull any ETH from the `ManagedWallet`.



### L-9
#### Control of `proposedConfirmationWallet` should be confirmed
`ManagedWallet::changeWallets()` allows the `proposedMainWallet` to be promoted to the `mainWallet` and the `proposedConfirmationWallet` is also promoted to the `cnfirmationWallet`,
but there has been no transaction from the `proposedConfirmationWallet` verifying it is controlled / correct.

If the `proposedConfirmationWallet` is not correct the primary function of `ManagedWallet` would be broken (being able to change the `mainWallet`).

##### Recommended Mitigation
Use a two-step approach, where after the `proposedMainWallet` confirms acceptance, the `proposedConfirmationWallet` may then also confirm, at which point the proposed wallets are promoted.



### L-10
#### Both proposed wallets can be the same
`ManagedWallet::proposeWallets()` does not check that the two wallets are unique, allowing the same wallet to be both the main and confirmation.
Having one wallet fill both roles, would defeat the purpose of using `ManagedWallet`, to provide security by separating the roles.

##### Recommended Mitigation
Add a check that the proposed wallets do not equal each other.



### L-11
#### Upkeep has a race condition that creates an economic disincentive
`Upkeep.performUpkeep` is an essential part of the protocol that operates on a decentralized economic incentive model, encouraging users to execute it by offering rewards. The reward relies on a combination of swap activity and time elapsed since the last upkeep was performed.

On the Ethereum mainnet, the order of transactions within a block is generally determined by the combination of gas price and miner tip. It's possible for multiple transactions to call the same function within the same block.

The design of `Upkeep.performUpkee`p allows it to be called multiple times within the same block. However, after the initial call in a block, subsequent calls won't receive any reward because there's no time gap since the previous upkeep.

This setup creates a situation where users might pay gas fees to fully process a transaction without receiving any reward if they don't win the transaction ordering race. This results in an inflated penalty for users who end up losing this contest.

##### Recommended Mitigation
In [Upkeep::performUpkeep()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L246)
```diff
	function performUpkeep() public nonReentrant
	{
+       require(lastUpkeepTimeEmissions != block.timestamp, "No time since elapsed since last upkeep"); 		
+
        // Perform the multiple steps of performUpkeep()
        try this.step1() {}
        catch (bytes memory error) { emit UpkeepError("Step 1", error); }		
```



### L-12 The first staker gets all preloaded rewards
A newly whitelisted token receives bootstrap rewards from the reward emitter as part of the upkeep cycle.

When rewards are emitted to a LP before any providers have staked, the first provider will gain all those initial rewards, while subsequent stakers in the same upkeep cycle receive none.

#### Proof of concept
This is due to [StakingRewards::_increaseUserShare()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L75-L92)
```solidity
		// Determine the amount of virtualRewards to add based on the current ratio of rewards/shares.
		// The ratio of virtualRewards/increaseShareAmount is the same as totalRewards/totalShares for the pool.
		// The virtual rewards will be deducted later when calculating the user's owed rewards.
        if ( existingTotalShares != 0 ) // prevent / 0
        	{
			// Round up in favor of the protocol.
			uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards[poolID] * increaseShareAmount, existingTotalShares );

			user.virtualRewards += uint128(virtualRewardsToAdd);
	        totalRewards[poolID] += uint128(virtualRewardsToAdd);
	        }

		// Update the deposit balances
		user.userShare += uint128(increaseShareAmount);
		totalShares[poolID] = existingTotalShares + increaseShareAmount;
```
When there are no `existingTotalShares`, then there are no `virtualRewardsToAdd`, which is the rewards accumulation counter. 

<details>
<summary>Test Case</summary>
As the same function is shared between liquidity and staking, `poolIDs[0]` is used for brevity of example.

Add the test to `src/staking/tests/StakingRewards.t.sol`
```solidity
    function testFirstStakerGetsReward() external {
        uint256 shareAlice = 5 ether;
        uint256 shareBob = 5 ether;
        uint256 rewards = 2 ether;

        // Assert clean start state
        assertEq(stakingRewards.userShareForPool(alice, poolIDs[0]), 0, "Alice's share should be zero");
        assertEq(stakingRewards.userShareForPool(bob, poolIDs[0]), 0, "Bob's share should be zero");
        assertEq(stakingRewards.totalShares(poolIDs[0]), 0, "Total shares should be zero");
        assertEq(stakingRewards.totalRewards(poolIDs[0]), 0, "Total rewards should be zero");
        assertEq(salt.balanceOf(address (stakingRewards)), 0, "Salt balance should be zero");

        // Add reward to the pools - before any users stake
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward(poolIDs[0], rewards);
        stakingRewards.addSALTRewards(addedRewards);

        assertEq(stakingRewards.totalRewards(poolIDs[0]), rewards, "Total rewards should be 2 ether");

        // Increase Alice's stake
        vm.startPrank(DEPLOYER);
        stakingRewards.externalIncreaseUserShare(alice, poolIDs[0], shareAlice, true);
        vm.stopPrank();

        assertEq(stakingRewards.userShareForPool(alice, poolIDs[0]), shareAlice, "Alice's share should have increased");
        assertEq(stakingRewards.totalShares(poolIDs[0]), shareAlice, "Total rewards should be sum of Alice's share");

        // @audit Alice has earned rewards despite staking AFTER the rewards were added to the pool
        assertEq(stakingRewards.userRewardForPool(alice, poolIDs[0]), rewards, "Alice has all the rewards despite staking AFTER they were added, not before");

        // Increase Bob's stake
        vm.startPrank(DEPLOYER);
        stakingRewards.externalIncreaseUserShare(bob, poolIDs[0], shareBob, true);
        vm.stopPrank();

        assertEq(stakingRewards.userShareForPool(bob, poolIDs[0]), shareBob, "Bob's share should have increased");
        assertEq(stakingRewards.totalShares(poolIDs[0]), shareAlice + shareBob, "Total rewards should be sum of Alice's and Bob's shares");
        assertEq(stakingRewards.userRewardForPool(bob, poolIDs[0]), 0, "Bob's share is zero");
        assertEq(stakingRewards.userRewardForPool(alice, poolIDs[0]), rewards, "Alice still has the rewards, even after totalShares increase!");
    }
```
</details>

#### Recommended Mitigation
Don't emit rewards until there is at least one staker providing LP.






## Non-Critical

### NC-1
#### When a user recasts their vote emit an event
When `Propsoals::castVote()` is used to recast a vote no event is emitted when the prior vote is removed, but there is one for newly cast vote.



### NC-2
#### Proposals::userHasActiveProposal() will be incorrect for the DAO
`Proposals::userHasActiveProposal()` determines if a given address has an active proposal. 
It operates under the assumption that only one open proposal per user is permitted, but the DAO is an exception and can have multiple open proposals, where this function may return `false` if queried after the closure of one proposal while others remain open. 
This is due to the function being set to return false upon the finalization of a proposal, not accounting for the possibility that the DAO could still have other ongoing proposals.



### NC-3
#### `Proposals::openBallots()` returns an unbounded set
No limit is imposed on the number of open proposals. `Proposals::openBallots()` copies the entire set of proposals to the memory return buffer, which can cause gas issues for any calling contract, but also data or timeout limits for JSON-RPC retrieval, due to overwhelming data volume.



### NC-4
#### Inconsistent quorum tiers between `minimumQuorum` and `requiredQuorum`
In `Proposals::requiredQuorumForBallotType()` when calculated tiered required quorum is below 5% of the total SALT, that is used, instead of a tiered required quorum amount.

There are three tiers for proposals, the lowest tier being for `PARAMETER` changes, the middle tier for `WHITELIST_TOKEN` and `UNWHITELIST_TOKEN`, and the highest tier for all other types. 
The quorum for the lowest tier is the `baseBallot` amount, for the middle tier: 2 * `baseBallot`, and for the top tier: 3 * `baseBallot`, displaying the principle that more significant the proposal the larger quorum required.

However, there's an issue with the minimumQuorum, which is a flat rate applied uniformly. When the quorums for all three tiers fall below this minimumQuorum, they are set to the same level, disregarding the different impact levels these tiers are supposed to represent in the protocol.



### NC-5
#### Integer rounding may leave dust in `Airdrop`
`Airdrop` calculates the `saltAmountForEachUser` based on the number of airdrop recipients.

[Airdrop::allowClaiming()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L62C1-L64C60)
```solidity
    saltAmountForEachUser = saltBalance / numberAuthorized();
```
Unless `saltBalance % numberAuthorized() == 0`, Salt dust will remain after a complete distribution to all claimants.



### NC-6
#### Hardcoded ETH amount for confirmation signal is brittle
`ManagedWallet::receieve()` requires the `confirmationWallet` to send at least `0.05 ether` to signal their agreement to the proposed wallet changes.

[ManagedWallet::receive()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L63C1-L68C60)
```solidity
    // Confirm if .05 or more ether is sent and otherwise reject.
    // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    if ( msg.value >= .05 ether )
        activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    else
        activeTimelock = type(uint256).max; // effectively never
```

While `0.05 ether` is only $125 USD at an Ether price of $2,500, if the dizzying heights of $10K+ are achieved the cost will eventually be non-trivial.



### NC-7
#### Governance update ro geo-restriction depends on off-chain private party
`Liquidty::_depositLiquidityAndIncreaseShare()` requires that users must have current exchange access deposit collateral. This requirement applies even if the user is trying to add collateral to their existing position, to increase their collateral health.

The ability to invalidate the exchange access of all users can be executed through a governance proposal. However, restoring access is contingent on an off-chain entity providing a signature (`SigningTools.EXPECTED_SIGNER`) to the user, who must then submit it to `AccessManager::grantAccess()`.



### NC-8
#### `Staking::unstakesForUser()` enumerates over an unbounded set
A user can increase the size of their set of active unstake by simply performing unstake and no limit is imposed.
`Staking::unstakesForUser()` enumerates every one of the user's active unstake actions, which can cause gas issues for any calling contract, but also data or timeout limits for JSON-RPC retrieval, due to overwhelming data volume.



### NC-9
#### `CollateralAndLiquidty::findLiquidatableUsers()` enumerates over an unbounded set
`CollateralAndLiquidty::findLiquidatableUsers()` enumerates over every holder borrower USDS to determine whether they are liquidate.

As there is no limit on the number of USDS borrowers, gas issues can be caused for any calling contract, but also data or timeout limits for JSON-RPC retrieval, due to overwhelming data volume.



### NC-10
#### Exchange can be started with only a single yes vote
`BootstapBallot::finalizeBallot()` does not have a required quorum, it only checks whether `startExchangeYes > startExchangeNo`, as a result the exchange would be started if there is a single yes vote, without any no votes. 
