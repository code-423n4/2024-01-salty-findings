0. `Proposals::proposeSendSALT()` - L202: Whenever salt balance of DAO.sol is `0 < DAO salt balance < 20`, `maxSendable` will round down to zero, therefore 0 salt will be sent even when salt balance is `19.99`.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/dao/Proposals.sol#L202

Whenever `exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) )` is `> 0 but < 20`, `maxSendable` will round down to zero, and therefore user passed parameter `amount` (which should be > 0) will trigger a revert in `require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );`, therefore wont be possible to send salt to anyone whenever DAO has `salt tokens > 0 but < 20`.

Impact:

- should be minimal on average
- approved proposal for sending salt reverting
- approved multi-proposal reverting due to salt balance less than 20, this could be more serious

PoC:

L202:
`maxSendable = balance * 5 / 100`
Will round down to zero for `balance * 5 / 100 < 1`:
`balance * 5 / 100 < 1`
`balance < 100/5`
`balance < 20`

So for DAO salt `balance < 20`, e.g. 19.99, `maxSendable` will round down to zero due to how solidity/evm treats non-integer numbers, especially anything less than 1.

Recommendation:

Might be OK if DAO salt balance will never be less than 20, otherwise:
- maybe leverage Fixed-Point Arithmetic library

=======
=======

1. `DAO::_executeApproval()` - L170: `( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )` should use `>` instead of `>=` because of the related check in `Proposals::proposeSendSALT()` at L203: `require( amount <= maxSendable`.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/dao/Proposals.sol#L203
https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/dao/DAO.sol#L170

`_executeApproval()` L170:
```solidity
if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
```

Should be `>` due to check in `Proposals::proposeSendSALT()` and in line with this comment too: "Only one sendSALT Ballot can be open at a time and the sending limit is 5% of the current SALT balance of the DAO." 

Impact:

- minimal, but more logically intact code, and in line with the correct logic in `Proposals::proposeSendSALT()` at L203.

Recommendation:

So, L170 should ideally be `<= salt.balanceOf(address(this)) * 5 / 100;` as follows:
```solidity
if ( exchangeConfig.salt().balanceOf(address(this)) * 5 / 100 >= ballot.number1 )
```

=======
=======

2. LOW: `ArbitrageSearch::_rightMoreProfitable()` - L68: potential rounding down to zero risk in `uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);`

Chances of this happening is small, but not impossible, should be some extreme market or other external event which causes the initial reserves of 1000 to drop to updated reserves of 20, and with midpoint of 50 would cause `amountOut` to be rounded down to zero, effectively causing user to lose their funds during the swap/trade.

IMPACT:

- high, user will lose their funds used in the swap/trade
- chance of happening pretty low, due to extreme conditions/manipulation required. flashloan maybe?

PoC:

```solidity
uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
```
For rounding down to zero we need the following condition to be true:
`reservesA1 * midpoint < reservesA0 + midpoint`
So:
= `reservesA1 < (reservesA0 + midpoint) / midpoint`
= `reservesA1 < (reservesA0 / midpoint) + 1`

Then when we apply this specific example:
`reservesA0 = 1000`, `reservesA1 = 20`, and `midpoint = 50`, the condition for rounding down to zero is met. However, such extreme reductions in reserves from 1000 to 20 are unrealistic in typical AMM scenarios and may indicate abnormal behavior, a potential issue, or an attack on the system.

Completing the above equation:
= `20 < 21`
So if we go back to the original equation for `amountOut`:
`amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint)`
= `(20 * 50) / (1000 + 50)`
= `0.952380952381`
which will be rounded down to:
= `0`

=======
=======

3. `SigningTools::_verifySignature()` - L26: Recommend to use OpenZeppelin's latest ECDSA over `ecrecover()` to enhance signature security and mitigate malleability risks.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/SigningTools.sol#L26

(Although it may not pose any risk for this specific implementation due to the fact that the signature is expected to be reused, the recommendation remains valid.)

L26:
```solidity
address recoveredAddress = ecrecover(messageHash, v, r, s);
```

RECOMMENDATION:

When it comes to smart contract signatures, consider swapping out `ecrecover()` for OpenZeppelin's ECDSA. Why? Well, certain signature schemes, like ECDSA, can get tricky with signature malleability, allowing the creation of distinct valid signatures for the same message and private key. OpenZeppelin's ECDSA library steps in as an advanced alternative, reducing the risk of encountering multiple valid signatures tied to a single message and key. In a nutshell, making this switch enhances signature security in smart contracts by addressing malleability challenges.

It's noteworthy that the missing check for a returned `address(0)` is not considered a concern, given the subsequent boolean "check" on L28 that effectively covers this scenario.

=======
=======

4. `SigningTools::_verifySignature()` - L13: The constant `65` in the require statement `require(signature.length == 65, "Invalid signature length");` is a magic number.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/SigningTools.sol#L13

Consider replacing it with a named constant or providing a comment explaining why the length is expected to be `65`.

=======
=======

5. QA: `CoreChainlinkFeed::latestChainlinkPrice()` - L32: Replace deprecated Chainlink functions like `latestRoundData()` with current recommended methods for accurate oracle data retrieval in smart contracts.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/price_feed/CoreChainlinkFeed.sol#L32

Using outdated Chainlink functions like `latestRoundData()` might serve up a bit of stale or wonky data, messing with the mojo of the contracts and price oracle functionality.

Reference:
https://github.com/code-423n4/2022-04-backd-findings/issues/17

Good effort by the dev to try implement mitigations against the issues caused by `latestRoundData()`:
https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/price_feed/CoreChainlinkFeed.sol#L26-L60

Recommendation:

Despite the dev's efforts to address concerns with the deprecated `latestRoundData` method, opting for the recommended, up-to-date Chainlink practices is advisable. This ensures a proactive and reliable approach, aligning with current standards and reducing reliance on outdated functionality for long-term stability.

=======
=======

6. QA: `PriceAggregator::setPriceFeed()` - L47: Risk of 35-day reliance on a single price feed if two feeds fail simultaneously, as `setPriceFeed()` can only be called once every 35 days for performance review.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/price_feed/PriceAggregator.sol#L47

Potential problem if 2 pricefeeds go down at similar time and become unreliable for longer than acceptable, then stuck with just 1 pricefeed for at least 35 days, unless emergency action is implemented for such a case:

Comment on L11:
`"setPriceFeed can only be called once every 35 days by default (to allow time to review performance of the most recently upgraded PriceFeed before setting another)."`

=======
=======

7. QA/LOW: `DAO::_executeSetContract()` - L133: Missing input validation check for zero hash: if for whatever reason `nameHash = 0x` then event `SetContract()` will still be emitted.

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/dao/DAO.sol#L131-L145

Due to missing input validation if `nameHash = 0x` then the `if` block will be skipped and successfully emit event `SetContract(ballot.ballotName, ballot.address1)`, which would be unexpected and potentially misleading, besides the fact that the function won't revert as it should and could potentially cause problems down the line.

Chances of `nameHash = 0x` happening is probably minimal but unless gas optimization is a valid excuse, the relevant input validation check should be implemented just to play it safe.

Recommendation:

```diff
	function _executeSetContract( Ballot memory ballot ) internal
		{
		bytes32 nameHash = keccak256(bytes( ballot.ballotName ) );
+		require(nameHash != 0x, "Invalid/zero hash");
```

=======
=======

8. QA: Throughout the in-scope contracts there are several state variables which might contain sensitive/private/confidential data that use the `private` modifier, which can be viewed by anyone.

Blockchain data, even information designated as `private` within smart contracts, is susceptible to visibility by individuals skilled in querying the blockchain's state or analyzing transaction history. Private variables, despite their designation, are not immune to public scrutiny.

Recommendation:

To address this potential issue, it is recommended to either store sensitive data off-chain or encrypt it before being stored on-chain. Extra care should be taken to prevent on-chain data from inadvertently exposing private information.

=======
=======

9. QA: `InitialDistribution::distributionApproved()` - L53: If for whatever reason `salt.balanceOf(address(this))` gets skewed from expected `100 * MILLION_ETHER`, will cause DoS of token distribution & permanent freezing of funds. 

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

=======
=======

10. `ManagedWallet::receive()` - L65: Contract intended to receive ETH via `receive()` but seems to be no way to ever get any ETH out again, which is problematic unless its deliberate intention of protocol team?

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

11. `Proposals::proposeSendSALT()` & `DAO::_executeApproval()` - lack of input validation for `amount` aka `ballot.number1` parameter, can pass zero value and result in 0 salt safeTransferred.

```solidity
function proposeSendSALT( address wallet, uint256 amount, string calldata description )
```
```solidity
function _executeApproval( Ballot memory ballot )
```

=======
=======
