# Gas Findings

## [GAS-1] `Airdrop`: Use Merkle Roots for Airdrops Instead of Adding Addresses One-by-one

You can save gas for the deployer by using a Merkle Root to determine eligibility for airdrops. The current way of adding `_authorizedUsers` one by one with `authorizeWallet(address)` is gas extensive. Or at least include a batched version of `authorizeWallet(address[])`.

### Link To Affected Code

https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/launch/Airdrop.sol#L46-L52