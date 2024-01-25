In Upkeep.sol's function performUpkeep, when step2() is called, msg.sender is the input paramter. For step2(), there is a modifier onlySameContract, which means require(msg.sender == address(this), so the input parameter for step2() should be address(this), ther receiver is in fact the contract itself address(this). Then the following culculation and safeTransfer are not needed at all. Even step2() is not needed at all.

https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L112-L123

function step2(address receiver) public onlySameContract
		{
		uint256 withdrawnAmount = exchangeConfig.dao().withdrawArbitrageProfits(weth);
		if ( withdrawnAmount == 0 )
			return;

		// Default 5% of the arbitrage profits for the caller of performUpkeep()
		uint256 rewardAmount = withdrawnAmount * daoConfig.upkeepRewardPercent() / 100;

		// Send the reward
		weth.safeTransfer(receiver, rewardAmount);
		}


https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L244-L251
	function performUpkeep() public nonReentrant
		{
		// Perform the multiple steps of performUpkeep()
 		try this.step1() {}
		catch (bytes memory error) { emit UpkeepError("Step 1", error); }

 		try this.step2(msg.sender) {}
		catch (bytes memory error) { emit UpkeepError("Step 2", error); }