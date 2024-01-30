# QA Report

## Low Severity Findings

### L-1 - `callContract` proposals are created with no `description`

## Impact

`callContract` proposals will be created with no `description`. The `Proposal` contract will be bricked, as it can't be fixed on-chain. Off-chain tools and integrations will have to create special code to lead with this error. Integrating contracts will have to take this error into account as well.

## Vulnerability Details

Note how `description` is placed on the `string1` parameter of [_possiblyCreateProposal](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81) for `callContract` proposals. `string2` is actually used for the `description`, so its actual value here will be `""`.

```solidity
    // Proposes calling the callFromDAO(uint256) function on an arbitrary contract.
    function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID) {
        require( contractAddress != address(0), "Contract address cannot be address(0)" );

        string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
👉      return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
    }
```

`string1` is used for calling attributes everywhere else, and `string2` is used instead for the `description`, like in [createConfirmationProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L126), [`proposeParameterBallot()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L158), [`proposeTokenWhitelisting()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L173) just to name a few, but it holds for all proposals.

## Recommended Mitigation Steps

Fix the code like this:

```diff
    // Proposes calling the callFromDAO(uint256) function on an arbitrary contract.
    function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID) {
        require( contractAddress != address(0), "Contract address cannot be address(0)" );

        string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
-       return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
+       return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, "", description );
    }
```

### L-2 - `withdrawArbitrageProfits()` will revert when `depositedWETH <= PoolUtils.DUST`

`withdrawArbitrageProfits()` will revert when attempting [to withdraw](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L304) less than the dust amount, [because of a check on the pool](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L222).

This is called [in step 2 of the Upkeep](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L114).

#### Recommendation

Safely return when the `depositedWETH` is less than the pools dust:

```diff
    // The arbitrage profits are deposited in the Pools contract as WETH and owned by the DAO.
    uint256 depositedWETH = pools.depositedUserBalance(address(this), weth );
-   if ( depositedWETH == 0 )
+   if ( depositedWETH <= PoolUtils.DUST )
        return 0;

    pools.withdraw( weth, depositedWETH );
```

### L-3 - `string1` and `string2` length is not validated when creating proposals

[In `_possiblyCreateProposal()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81) `string1` length is not validated for `tokenIconURL` or `newWebsiteURL`. `string2` is not validated for the proposal `description`.

Very large strings will make proposals difficult to read for voters.

#### Recommendation

Set a limit to the length of the strings

### L-4 - It is possible to create proposals that can't be finalized

Certain attributes make passing proposal impossible to be finalized, as they will revert. This also bricks the ability of the proposer to ever propose another ballot again, and prevents other users from creating a new ballot with the same name.

Affected proposals:

- `proposeParameterBallot()` allows to [set any `parameterType`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L158), but it can revert when trying to [convert it to `ParameterTypes`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L120), if the number is ot of bounds [for the `ParameterTypes`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Parameters.sol#L14-L53) enum.

#### Recommendation

Verify the range of the attributes on proposal creation.

### L-5 - `tokenWhitelistingBallotWithTheMostVotes()` returns the ballot id `0` when no proposal reaches quorum

[If no proposal votes reaches quorum](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L429), then the function will [returns its default value, which is `0`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L421).

#### Recommendation

Revert when the returned value is `0`. It is safe, as [`ballotId` starts at 1](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L108), and when the function is used to [finalize whitelisted tokens](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L250), the ballot already needs to reach quorum to get to this point.

### L-6 - `websiteURL` should be set in the constructor

The `websiteURL` [is set on the DAO](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L150) only [after a ballot voting passes](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L214). The initial setup will take many days, from airdrop, to staking, to proposing, to voting, leaving the DAO contract with no initial website URL during that period.

#### Recommendation

Set an initial value for `websiteURL` in the `DAO` `constructor`.

### L-7 - Votes can't be delegated

The protocol is missing some function to delegate votes to other users.

This is important as not all users will participate in DAO voting (as there is no direct incentive), and those votes could be delegated to other users they trust and will be actively participating in the DAO. 

This makes it easier to reach quorum, to prevent malicious proposals from passing, and incentivizes participation of dedicated users in the DAO.

### L-8 - `proposeWallets` enters in a deadlock if the proposed wallet doesn't call `changeWallets()`

The `mainWallet` has the ability to propose new wallets, [which are tracked in storage](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L51-L52).

Confirmation wallets have the ability to confirm or reject those proposed wallets. When there is a confirmation, an [active timelock is set](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L66).

[Once the timelock is over, the proposed wallet](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L76-L77) can call `changeWallets()` to make the changes for real.

The problem is that if the proposed wallet doesn't call the function, or can't do it, it's not possible for the proposer to propose a new one, [because of this requirement](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L49).

#### Recommendation

Allow the proposer to reset the timelock, and the proposed addresses if the proposed wallet didn't call `changeWallets()` after some time.

## Non-Critical Issues

### L-9 - Replace `==` balance comparison with `>=`

Even if the initial distribution is supposed to have an exact number of tokens, it is safer to check that the [function to distribute them](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L50) won't revert if for any reason it has more tokens, which would lock them in the contract:

```diff
-   require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
+   require( salt.balanceOf(address(this)) >= 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```

[InitialDistribution.sol#L53](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L53)

### L-10 - Signatures do not implement EIP-712

#### Impact

[EIP-712 Motivation](https://eips.ethereum.org/EIPS/eip-712#motivation) is to improve the usability of off-chain message signing for use on-chain.

By not adhering to it, users will find more difficulty understanding what they are signing, as off-chain tools will have trouble decoding the messsage, leading to a compatibility and integration issue.

Signatures can also be replayed on other contracts, as the contract address is not included in the message hash.

#### Proof of Concept

Message hashes are implemented without following specs by EIP-712, missing the [`typeHash`](https://eips.ethereum.org/EIPS/eip-712#rationale-for-typehash), and the [`domainSeparator`](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator).

```solidity
bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
return SigningTools._verifySignature(messageHash, signature);
```

[AccessManager.sol#L53](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/AccessManager.sol#L53)

```solidity
bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );
```

[BootstrapBallot.sol#L53-L54](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L53-L54)

#### Recommendation

Follow the EIP-712 spec, including the corresponding `typeHash`, `domainSeparator`, contract address, and build the message hash accordingly.

### L-11 - DUST value too big for tokens with low decimals

Some tokens with a considerable market cap like [GUSD](https://coinmarketcap.com/currencies/gemini-dollar/), or [EURS](https://coinmarketcap.com/currencies/stasis-euro/) have 2 decimals.

The [`DUST`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L13) value is fixed at `100`, which can be considerable big as dust, since it is used all over the protocol, with many `require` depending on the amount being bigger than it.

#### Recommendation

Consider defining dust values considering proportional to the decimals of the tokens

### L-12 - Arbitrage profit is not calculated correctly when there is an index with an `INVALID_POOL_ID`

#### Impact

1/3 to 2/3 of arbitrage profit may not be distributed

#### Proof of Concept

Arbitrage profit is always divided by 3, despite some indicies might have an `INVALID_POOL_ID`. That means if an index has an invalid pool id, that 33% of arbitrage profits will not be distributed.

```solidity
    // Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
👉  uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
    if ( arbitrageProfit > 0 )
        {
        ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];

        if ( indicies.index1 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index1] += arbitrageProfit;

        if ( indicies.index2 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index2] += arbitrageProfit;

        if ( indicies.index3 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index3] += arbitrageProfit;
        }
    }
```

[PoolStats.sol#L111-L126](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L111-L126)

#### Recommendation

Count the number of indicies without an `INVALID_POOL_ID`, and then distribute the profit evenly among them

### L-13 - Users from excluded countries, without wallet authorization can call `cancelUnstake()`

There is no restriction in [`cancelUnstake()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L84-L97) to prevent wallets without access to call the function, like in [`stakeSALT()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L43).

Canceling the unstake process puts back the staked SALT in the contract, like with the staking process.

#### Recommendation

Consider not allowing unauthorized users to cancel their unstake process.

```diff
    function cancelUnstake( uint256 unstakeID ) external nonReentrant
+       require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
```

### L-14 - `sum` in `RewardsEmitter` is not used

The [`sum`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L115) variable is [calculated](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L128) in the `RewardsEmitter` but never used, nor returned.

It may have been left from a previous refactor, as a `sum` variable [is also used in `StakingRewards::addSALTRewards()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L201-L204), which is called by the `RewardsEmitter`.

The `RewardsEmitter` already [approved the SALT token to its maximum](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L50), so there should be no issue. On any case, I'm highlighting this, as it may hide some other issue I couldn't detect.

#### Recommendation

Verify that `sum` is actually not used, and remove it from the `RewardsEmitter` contract if not.

### NC-1 - `tokenIconURL` is not used in `proposeTokenUnwhitelisting()`

[`tokenIconURL`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180) is only used when whitelisting tokens but not when [unwhitelisting](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L157-L164).

#### Recommendation

Consider removing it from `proposeTokenUnwhitelisting()`