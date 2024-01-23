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
=======
=======

1. `ManagedWallet::receive()` - L65: Contract intended to receive ETH via `receive()` but seems to be no way to ever get any ETH out again, which is problematic unless its deliberate intention of protocol team?

- .05 ether isnt a small amount, currently it's around $125, so when this is sent to this contract 10 times, it's already $1250 of ETH funds stuck forever in the contract...

- I checked and didn't see any methods/functions to withdraw the ETH funds again...

```solidity
	// The confirmation wallet confirms or rejects wallet proposals by sending a specific amount of ETH to this contract
    receive() external payable
    	{
    	require( msg.sender == confirmationWallet, "Invalid sender" );

		// Confirm if .05 or more ether is sent and otherwise reject.
		// Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    	if ( msg.value >= .05 ether )
    		activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    	else
			activeTimelock = type(uint256).max; // effectively never
        }
```
RECOMMENDATION:

- add withdraw method with access control so that admin/owner can withdraw the ETH if ever necessary.

via `transfer()`:
```solidity
    // Function to allow the owner to withdraw ETH from the contract
    function withdraw(uint256 amount) external onlyOwner {
        require(amount > 0, "Withdrawal amount must be greater than 0");
        require(address(this).balance >= amount, "Insufficient balance in the contract");

        // Transfer ETH to the owner
        payable(owner).transfer(amount);

        // Emit event
        emit Withdrawal(owner, amount);
    }
```
OR

via `call()`:
```solidity
function withdraw(uint256 amount) external onlyOwner {
    require(amount > 0, "Withdrawal amount must be greater than 0");
    require(address(this).balance >= amount, "Insufficient balance in the contract");

    // Call with explicit gas and value
    (bool success, ) = payable(owner).call{value: amount, gas: gasleft()}("");
    require(success, "Transfer failed");

    // Emit event
    emit Withdrawal(owner, amount);
}
```
=======
=======

