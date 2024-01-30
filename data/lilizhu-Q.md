## [L-01] **Missing address(0) checks in** ManagedWallet contract

### Impact

In the constructor of the ManagedWallet contract in the src/ManagedWallet.sol file, there is no verification of non-zero addresses for the input parameters. If the product team specifies _mainWallet or _confirmationWallet as address(0) during initialization, it will cause the entire system's mainWallet mechanism to completely fail.

- If _mainWallet is mistakenly specified as address(0) during initialization, no one can propose a new main wallet and confirmation wallet, causing the mainWallet mechanism to fail.
- If _confirmationWallet is mistakenly specified as address(0) during initialization, no one can confirm or reject the confirmation wallet, causing the mainWallet mechanism to fail.

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L31

ps: I don't agree with the reasons described   “**The absence of checks for the zero address ('0x0') in the code is an intentional gas-saving measure.”**,because the construct function is more import than proposeWallets() function, the proposeWallets() function also has the address(0) check, so the construct function more necessary set the address(0) checker.

## Tools Used

vs code, forge

## Recommended Mitigation Steps

```solidity

constructor( address _mainWallet, address _confirmationWallet)
		{
		require( _mainWallet != address(0), "_mainWallet cannot be the zero address" );
		require( _confirmationWallet != address(0), "_confirmationWallet cannot be the zero address" );

		mainWallet = _mainWallet;
		confirmationWallet = _confirmationWallet;

                // Write a value so subsequent writes take less gas
		activeTimelock = type(uint256).max;
        }
```