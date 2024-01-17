# Gas Findings

## [GAS-1] `Airdrop`: Use Merkle Roots for Airdrops Instead of Adding Addresses One-by-one

You can save gas for the deployer by using a Merkle Root to determine eligibility for airdrops. The current way of adding `_authorizedUsers` one by one with `authorizeWallet(address)` is gas extensive. Or at least include a batched version of `authorizeWallet(address[])`.

### Link To Affected Code

https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/launch/Airdrop.sol#L46-L52

## [GAS-2] `ManagedWallet`: Do not Burn 0.05ETH Just to Confirm a Change in Ownership

The ETH sent to `ManagedWallet` is effectively lost since there's no functions implemented to retrieve it. This basically makes the gas cost of changing ownership 0.05ETH more than it's needed.

Instead, you can just send the ETH back to the confirmation wallet, even if the `confirmationWallet` is "custodial". However, it is highly advised the team to not ever use a wallet that cannot submit calls as a `confirmationWallet` for anything. If this limitation is relaxed it's best to redo the implementation so no ETH needs to be sent at all.

### Improved Function:

```
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

  

        payable(msg.sender).transfer(msg.value);

        }
```

### Link To Affected Code

https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/ManagedWallet.sol#L59-L69

## [GAS-3] `ManagedWallet`: `activeTimelock` Don't Need to be Updated for a Rejection

You can remove `activeTimelock = type(uint256).max;` for a rejection since `activeTimelock` will already be at `uint256.max` (either due to the constructor starting state or because `changeWallets()` changed it to `uint256.max` before the proposal is made).

### Improved Function:

```
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

  

        payable(msg.sender).transfer(msg.value);

        }
```

### Link To Affected Code

https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/ManagedWallet.sol#L59-L69