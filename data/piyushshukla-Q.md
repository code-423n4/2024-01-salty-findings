StakingRewards smart contract, specifically in the unstake function. The issue stems from the absence of validation for the numWeeks parameter, allowing users to execute unstaking operations with a duration of zero weeks or a negative value.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L60

        function unstake( uint256 amountUnstaked, uint256 numWeeks ) external nonReentrant returns (uint256 unstakeID)
                {
                require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" );


                uint256 claimableSALT = calculateUnstake( amountUnstaked, numWeeks );
                uint256 completionTime = block.timestamp + numWeeks * ( 1 weeks );


                unstakeID = nextUnstakeID++;
                Unstake memory u = Unstake( UnstakeState.PENDING, msg.sender, amountUnstaked, claimableSALT, completionTime, unstakeID );


## fix

    require(numWeeks > 0, "Invalid unstake duration");

