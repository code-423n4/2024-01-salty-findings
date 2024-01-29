## Impact
[L-1] - stakeSALT should add check for minimum staking amount.
[stakeSALT](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L41-L53)


[L-2] - use safeerc20.safeapprove instead of approve
since SafeERC20 was imported then its better to use safeapprove instead of approve.

On these contracts: 
./dao/DAO.sol
./stable/Liquidizer.sol
./Upkeep.sol
./launch/Airdrop.sol
./staking/Liquidity.sol
./rewards/RewardsEmitter.sol
./rewards/SaltRewards.sol

## Recommended Mitigation Steps
change approve to safeapprove






