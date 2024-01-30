## [L-01] Parameter Ballot can be proposed with invalid `parameterType`

If [`Proposals.proposeParameterBallot(...)`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L155-L159) is called with an invalid `parameterType` the function does not revert. I am reporting this as a LOW severity because it does not break the protocols functionality as an invalid `parameterType` does not do anything.

## Suggestion

Modify the `Proposals.proposeParameterBallot(...)` as shown below

```
function proposeParameterBallot( uint256 parameterType, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
+		require(parameterType < 26, "Invalid ParameterType");
		string memory ballotName = string.concat("parameter:", Strings.toString(parameterType) );
		return _possiblyCreateProposal( ballotName, BallotType.PARAMETER, address(0), parameterType, "", description );
		}
```


## [L-02] `SendSALT` can be proposed with zero amount

If [`Proposals.proposeSendSALT(...)`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L196-L209) is called with zero as the `amount` of SALT to send, the function does not revert.

Although no SALT is sent even when the proposal passes but the caller still spends gas fee when the call is made.

## Suggestion

- Modify the `Proposals.proposeSendSALT(...)` to revert when the SALT balance is less than `100 wei` (to avoid rounding errors)

```solidity
function proposeSendSALT( address wallet, uint256 amount, string calldata description ) external nonReentrant returns (uint256 ballotID)
		{
		...
		uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
		require(balance > 100 wei, "Insufficient SALT balance");
		uint256 maxSendable = balance * 5 / 100;

		...
		}
```
