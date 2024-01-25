## Missing check in ManagedWallets
`ManagedWallet` contract introduces 2 wallets `mainWallet` and `confirmationWallet`. It is possible to pick them as same addresses. Since they have different roles they must not be same. Consider adding relevant checks both in `constructor` and `proposeWallets()` to block the logic fail.
