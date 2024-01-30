# 1, User might unintentionally vote for other ballot when reorg happen

## Link to code
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L108C3-L108C29

## Vulnerability details
When creating ballot, ballotId is created with linear number pattern:

       ballotID = nextBallotID++;
It will lead to scenario that when user create proposal and vote for it, when reorg happen, attacker can create malicious ballot with same type and receive that vote. But since user can undo vote, and it require ballot need to pass minimum time to be able to be approved, along with chance that reorg happen in the mainnet is very small, so it is very hard to make this attack success

## Mitigration steps
BallotId should be generated by hashing proposer address and `ballotId`

--------------------------

# 2, approve() in `_depositLiquidityAndIncreaseShare()` does not reset to 0 after  transfer, future approve can be reverted

## Link to code
https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L83-#L117

## Vulnerability details
To make `Pools` contract able to use token in the contract, approve() is used:

		tokenA.approve( address(pools), maxAmountA );
		tokenB.approve( address(pools), maxAmountB );
But it might not use all of them, so the unused part will be refunded to the user:

		if ( addedAmountA < maxAmountA )
			tokenA.safeTransfer( msg.sender, maxAmountA - addedAmountA );

		if ( addedAmountB < maxAmountB )
			tokenB.safeTransfer( msg.sender, maxAmountB - addedAmountB );
Some tokens, like USDT, will revert new approve if allowance is not zero, it will lead to function revert.

## Mitigration steps
Approve to zero after calling `pools.addLiquidity()` function