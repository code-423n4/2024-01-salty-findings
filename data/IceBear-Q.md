## [L-01] Hardcoding EXPECTED_SIGNER reduces future scalability.
https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L7
```
address constant public EXPECTED_SIGNER = 0x1234519DCA2ef23207E1CA7fd70b96f281893bAa;
```
## [L-02] No mechanism for blacklisting.

Once a user has been granted permission, there is no mechanism for removal in the absence of a blacklist.
https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L65
```
// Grant access to the sender for the given geoVersion.
// Requires the accompanying correct message signature from the offchain verifier.
function grantAccess(bytes calldata signature) external
    {
        require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );

        _walletsWithAccess[geoVersion][msg.sender] = true;

        emit AccessGranted( msg.sender, geoVersion );
    } 
```

## [L-03] Excess tokens approval will be give to  the staking contract.
This issue is covered in the [bot race report](https://github.com/code-423n4/2024-01-salty/blob/main/bot-report.md#l-16) but lacks a more detailed explanation.

saltBalance is the  salt balance of the Airdrop contract.
saltAmountForEachUser is calculated as follows:

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L64
```
saltAmountForEachUser = saltBalance / numberAuthorized();
```
Afterward, approve saltBalance to the staking contract.
https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L67
```
salt.approve( address(staking), saltBalance );
```
claimAirdrop() only sends out saltAmountForEachUser.

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L81-L82
```
// Have the Airdrop contract stake a specified amount of SALT and then transfer it to the user
		staking.stakeSALT( saltAmountForEachUser );
		staking.transferStakedSaltFromAirdropToUser( msg.sender, saltAmountForEachUser );
```
Consider the following scenarios:
If saltBalance = 100
saltAmountForEachUser = saltBalance / numberAuthorized()
=> 100/8 = 12
Approved 4 salt in excess.
If saltBalance = 200
saltAmountForEachUser = saltBalance / numberAuthorized()
=> 200/26 = 182
Approved 18 salt in excess.

## [L-04] setContracts can be called multiple times.
setContracts can only be be called once
But when dao is the zero address, it can bypass the check.
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51
```
require( address(dao) == address(0), "setContracts can only be called once" );
```
The owner can set the address of many parameters multiple times.
Describe some of the impacts here.

- The airdrop address is expected to be set only once.
When the airdrop address can be reset.
This will affect functions that use the return value of walletHasAccess() to verify legitimacy, including [CollateralAndLiquidity.sol borrowUSDS()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L95), [Liquidity.sol _depositLiquidityAndIncreaseShare()](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L83), and [Staking.sol stakeSALT()](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L41).

- The upkeep address is expected to be set only once.
When the upkeep address can be reset.
can use [withdrawArbitrageProfits() from DAO.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L295) to withdraw the WETH arbitrage profits.
It also affects all functions that check permissions using ```msg.sender == address(exchangeConfig.upkeep)```.

## [L-05]The vote is ineffective.
Votes can still be cast after the completionTimestamp.

https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L48
```
// Cast a YES or NO vote to start up the exchange, distribute SALT and establish initial geo restrictions.
	// Votes cannot be changed once they are cast.
	// Requires a valid signature to signify that the msg.sender is authorized to vote (being whitelisted and the retweeting exchange launch posting - checked offchain)
	function vote( bool voteStartExchangeYes, bytes calldata signature ) external nonReentrant
		{
		require( ! hasVoted[msg.sender], "User already voted" );

		// Verify the signature to confirm the user is authorized to vote
		bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
		require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );

		if ( voteStartExchangeYes )
			startExchangeYes++;
		else
			startExchangeNo++;

		hasVoted[msg.sender] = true;

		// As the whitelisted user has retweeted the launch message and voted, they are authorized to the receive the airdrop.
		airdrop.authorizeWallet(msg.sender);
		}
```

finalizeBallot allows anyone to call.
And finalizeBallot can be called immediately once completionTimestamp is reached.
Users can observe the voting process and immediately front-run favorable outcomes for themselves once the completionTimestamp is reached.
Votes cast by users after this point will be ineffective.
This could impact credibility.
