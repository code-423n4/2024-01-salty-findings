
# 1. Consider checking for similarities in current and proposed wallets
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L42
## Impact
Improves decentralization, prevents loss of funds needed to accept proposals in case same wallet updates (Updating wallets to the same addresses requires sending ether to accept, or else new wallet proposals cannot be made). 

## Recommended Mitigation Steps
Consider including checks
proposedMainWallet != proposedConfirmationWallet
mainWalllet != proposedMainWallet
confirmationWallet != proposedConfirmationWallet

***


# 2. Protocol is highly vulnerable to WBTC pausable property. 
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L70
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L140
https://etherscan.io/token/0x2260fac5e5542a773aa44fbcfedf7c193bc2c599#code
## Impact

The protocol relies in WBTC as one of the main collateral tokens for the protocol. Users looking to repay their USDS debt, or improve their collateral share, might need to deposit more WBTC/WETH collateral. 

This process fails if WBTC is paused as transfers will not be executable thus causing such borrowers to fall below the required liquidation threshold.

At the other end, liquidators will also not be able to liquidate these users as the wbtc liquidity removal from the pools will fail. This will cause a host of bad debt to be accrued up in the protocol.

Whenever the token gets unpaused, liquidators using MEV bots can also frontrun the borrowers' repayments leading to a host of unfair liquidations. 

## Recommended Mitigation Steps
Recommend introducing support for other types of tokens, to serve as collateral for USDS in the `CollateralAndLiquidity` contract. A more standard ERC20 token can be used. In cases where WBTC is paused, the liquidation and debt repayment process can be conducted using these alternative tokens. 

To prevent unfair liquidations after WBTC unpause, a short downtime period can be put in place on the liquidate function, to allow borrowers to execute their repayments first.


***

# 3. Use of unsafe signature verification method
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/SigningTools.sol#L26
## Impact

The built-in EVM pre-compiled `ecrecover` is susceptible to signature malleability due to non-unique v and s (which may not always be in the lower half of the modulo set) values, possibly leading to replay attacks. Elsewhere if devoid of the adoption of nonces, this could prove a vulnerability when not carefully used.
The method can also return a null address (address(0)). This can happen when the recovery ID (v) is set to any value other than 27 or 28 which may allow EXPECTED_SIGNER spoofing.
## Recommended Mitigation Steps
Consider - 
Adding a check for v to be either 27 or 28.
Adding a check for signer != address(0)
Using OpenZeppelin’s ECDSA library.


***

# 4. Check for poolID existence/whitelist status 
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L212
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L222
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L256
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L273
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L290
## Impact
`totalRewardsForPools`, `totalSharesForPools`, `userRewardsForPools`, `userShareForPools`, `userCooldowns`
These functions feature unbounded loops and take in poolIDs without checking for their existence or whitelist status. Thus, a malicious attacker can pass in a large array of non-existent poolId which can be used to DOS the contract.
## Recommended Mitigation Steps
Adding a check for poolID whitelist status, severly limts the amount of invalid pools that can be passed in. Other checks include capping the maximum amount of poolIds that can be check and preventing duplicate entries. 

***

# 5. `finalizeBallot` can be frontrun to delay bootstrap ballot outcome.
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L69
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L48
## Impact
There's no limit to how long a ballot can go on. The `finalizeBallot` can only be called after the completion time has passed. 

The `vote` function however doesn't impose any time bound on the voting process. Voters can delay the ballot finalization process by frontrunning the `finalizeBallot` function. 

In a rare case, they can change voting results after the ballot duration has passed, which is unfair to other users. 
## Recommended Mitigation Steps
Include a check for ballot duration `vote` function. 
		require( block.timestamp < completionTimestamp, "Voting period over" )
***

***

# 6. Not all chainlink feeds returns 8 decimals.
Links to affected code *
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/CoreChainlinkFeed.sol#L59
## Impact
- The coreChainlinkFeed contract converts the returned prices from chainlink fron 8 to 18 decimals. However this assumes that all feeds return 8 decimals. Most ETH feeds return 18 decimals instead and some USD feeds too. 
In case a new token is whitelisted for the pools and the price returned will be inflated due to the conversion.
```
// Convert the 8 decimals from the Chainlink price to 18 decimals
		return uint256(price) * 10**10;
```

## Recommended Mitigation Steps
Check for returned decimals first instead of assuming 8.
