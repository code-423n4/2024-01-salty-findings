# Gas Efficiency Report

## Summary:

The gas report highlights potential optimizations to reduce gas consumption in the codebase. Instances in [Proposals.sol #90, #324, #326, #329] involve on-chain computations for known results, suggesting gas savings by avoiding unnecessary calculations. Furthermore, [Dao.sol #121] indicates a redundant second `if` check that can be removed for more efficient gas usage. Additionally, [Dao.sol #345] and [StakingRewards.sol #253] suggest areas where using `unchecked` could enhance gas efficiency while maintaining safety. These optimizations aim to mitigate gas costs during contract execution.

### Summary Table:

| Number | Title                                           | Number of Instances |
|--------|-------------------------------------------------|---------------------|
| [G-01]      | Avoid on-chain computation for known results     | 6                   |
| [G-02]      | Eliminate redundant second `if` check           | 1                   |
| [G-03]      | Use `unchecked` for safe gas optimization       | 2                   |

## [G-01] Avoid on-chain computation if the result is known 
### Instances
* [Proposals.sol #90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L90)
```solidity
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

```
* [Proposals.sol #324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324)
```solidity
			requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

```
* [Proposals.sol #326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326)
```solidity
			requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

```
* [Proposals.sol #329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329)
```solidity
			requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );

```
* [DAOConfig.sol #102](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L102)
```solidity
			if (baseBallotQuorumPercentTimes1000 < 20 * 1000)

```

* [DAOConfig.sol #107](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L107)
```solidity
			if (baseBallotQuorumPercentTimes1000 > 5 * 1000 )

```



## [G-02] No need for second if check
### Instances
* [Dao.sol #121](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L121)
```solidity
		else if ( winningVote == Vote.DECREASE )

```

## [G-03] Safe to Surround by unchecked
### Instances
* [Dao.sol #345](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L345)
```solidity
		uint256 remainingSALT = claimedSALT - amountToSendToTeam;

```
* [StakingRewards.sol #253](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L253)
```solidity
		return rewardsShare - user.virtualRewards;

```