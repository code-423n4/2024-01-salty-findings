# QA Report for Salty.io

## Low Risk 

### L-1 Staking configuration changes only applies to new unstake after finalization
Proposals to the `StakingConfig` alter the conditions for users wishing to unstake. 
Users who are already unstaking at the time of these changes will not be affected by them if the changes are advantageous, such as a decrease in the time required to unstake their complete share.

#### Recommended Mitigation
Include the `maxUnstakeWeeks` in the `Unstake` struct and provide a function for users to call. to reduce their staking time when the criteria have been altered with governance.



### L-2 Whitelisting, un-whitelisting then re-whitelisting to game bootstrap rewards
In the DAO governance process, when a token is approved through a proposal to be whitelisted, it becomes eligible for the `bootstrappingRewards` in SALT. 
There are no safeguards on the `bootstrappingRewards` for the same token being whitelisted again after it has been un-whitelisted, 
allowing the exploited of repeatedly listing and un-whitelisting tokens in order to artificially inflate the rewards received by the pool.

#### Recommended Mitigation
Keep a mapping of un-whitelisted tokens, and only provide `bootstrappingRewards` to no in that mapping.



### L-3 Price feed proposal are not check to ensure feed uniqueness
There is no verification to confirm that when the price feed from a proposal is applied, the three price feed addresses are distinct. 
If two or more of these feeds were directed to the same address, it would undermine the purpose of having a redundant feed system. As the two identical feeds would produce agreeing results, eliminating the benefit of having multiple sources for price information.

#### Recommended Mitigation
In `DAO::_executeSetContract()`, before applying the mutation check the three price feeds would remain unique.



### L-4 Rounding down prevents proposals when there is between 1 and 999 xSALT
With the lowest required proposal stake configuration, if the sum of staked SALT is between 1 and 999 xSALT then no proposals can be made, even though the proposer may have xSALT.

In [Proposals::_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L90-L92) 
```solidity
    uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

    require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
```
`requiredXSalt` is zero, due to the denominator being larger than the numerator.

#### Recommended Mitigation
If expected behaviour then document in NatSpec, otherwise use a default minimum value for `requiredXSalt` in a similar approach to required quorum. 



### L-5 When xSalt is low, there is a risk of proposal spam
The threshold for `requiredXSalt` is calculated using the current amount of xSalt (staked amount of SALT).

In [Proposals::_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L89-L91)
```solidity
			uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
```

When xSalt is low, the `requiredXSalt` will be even lower (`daoConfig.requiredProposalPercentStakeTimes1000()` ranges from 0.1% to 2%), with a lower bar cost to create a spam proposal (or self-interested one) more will get created.

#### Recommended Mitigation
Implement a default minimum value for `requiredXSalt` in a similar approach to required quorum, where if `requiredXSalt` is below the minimum use that instead.




## Non-Critical

### NC-1 When a user recasts their vote emit an event
When `Propsoals::castVote()` is used to recast a vote no event is emitted when the prior vote is removed, but there is one for newly cast vote.



### NC-2 Proposals::userHasActiveProposal() will be incorrect for the DAO
`Proposals::userHasActiveProposal()` determines if a given address has an active proposal. 
It operates under the assumption that only one open proposal per user is permitted, but the DAO is an exception and can have multiple open proposals, where this function may return `false` if queried after the closure of one proposal while others remain open. 
This is due to the function being set to return false upon the finalization of a proposal, not accounting for the possibility that the DAO could still have other ongoing proposals.



### NC-3 Proposals::openBallots() returns an unbounded set
No limit is imposed on the number of open proposals. `Proposals::openBallots()` copies the entire set of proposals to the memory return buffer, which can cause gas issues for call contracts, but also data or timeout limits for JSON-RPC retrieval, due to overwhelming data volume.



### NC-4 Inconsistent quorum tiers between `minimumQuorum` and `requiredQuorum`
In `Proposals::requiredQuorumForBallotType()` when calculated tiered required quorum is below 5% of the total SALT, that is used, instead of a tiered required quorum amount.

There are three tiers for proposals, the lowest tier being for `PARAMETER` changes, the middle tier for `WHITELIST_TOKEN` and `UNWHITELIST_TOKEN`, and the highest tier for all other types. 
The quorum for the lowest tier is the `baseBallot` amount, for the middle tier: 2 * `baseBallot`, and for the top tier: 3 * `baseBallot`, displaying the principle that more significant the proposal the larger quorum required.

However, there's an issue with the minimumQuorum, which is a flat rate applied uniformly. When the quorums for all three tiers fall below this minimumQuorum, they are set to the same level, disregarding the different impact levels these tiers are supposed to represent in the protocol.



