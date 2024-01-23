0. QA: `InitialDistribution::distributionApproved()` - L53: If for whatever reason `salt.balanceOf(address(this))` gets skewed from expected `100 * MILLION_ETHER`, will cause DoS of token distribution & permanent freezing of funds. 

- Due to the fact that salt tokens dont exist in public before the airdrop & salt token distribution, it would be near impossible for an attacker to send non-existent salt tokens to this contract in order to DoS the distribution functionality and permanently freeze the salt token funds in there. 
- However, there's likely no guarantee that the `salt.balanceOf(address(this))` won't get slightly skewed anyway due to external factors, so it would be wise to implement guaranteed mitigation against this potential scenario...

- Therefore this is at least a QA and at most a medium severity issue.
- IF there was any salt tokens already in circulation before this distribution function was called, this would be a high/critical severity vulnerability...

L53:
```solidity
require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```

IMPACT:

- no impact, unless the `salt.balanceOf(address(this))` was somehow skewed before token distribution, which is highly unlikely but not impossible. To make it impossible, the recommended "fix" should be implemented.

RECOMMENDATION:

- to change from `==` to `>=` to ensure the funds can never become stuck...
- always possible to transfer more salt tokens to this contract(if ever necessary), therefore the suggested modification below should suffice to permanently squash this vulnerability.

```diff
- require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
+ require( salt.balanceOf(address(this)) >= 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```

And then just modify/update the implemented logic where necessary to account for the above change, so that the intended logic behind the error message remains respected.

Function in question:
```solidity
    // Called when the BootstrapBallot is approved by the initial airdrop recipients.
    function distributionApproved() external
    	{
    	require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
		require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );

    	// 52 million		Emissions
		salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );

	    // 25 million		DAO Reserve Vesting Wallet
		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );

	    // 10 million		Initial Development Team Vesting Wallet
		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );

	    // 5 million		Airdrop Participants
		salt.safeTransfer( address(airdrop), 5 * MILLION_ETHER );
		airdrop.allowClaiming();

		bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

	    // 5 million		Liquidity Bootstrapping
	    // 3 million		Staking Bootstrapping
		salt.safeTransfer( address(saltRewards), 8 * MILLION_ETHER );
		saltRewards.sendInitialSaltRewards(5 * MILLION_ETHER, poolIDs );
    	}
```