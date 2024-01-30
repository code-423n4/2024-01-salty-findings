# proposeCallContract: Incorrectly stores the description

## Links to affected code

[https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/dao/Proposals.sol#L218](https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/dao/Proposals.sol#L218)

## Impact

It incorrectly stores the description. This may prevent the description from being retrieved offchain.

## Proof of Concept

```solidity
function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID)
{
    require( contractAddress != address(0), "Contract address cannot be address(0)" );

    string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
@>  return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
}
```

When calling `_possiblyCreateProposal`, it passes the `description` as the `string1` parameter. An empty string is stored as description. Therefore, the description may not appear when retrieved offchain.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Pass the `description` as the `string2` parameter.

```diff
function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID)
{
    require( contractAddress != address(0), "Contract address cannot be address(0)" );

    string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
-   return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
+   return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, "", description );
}
```

# withdraw - Can leave less than DUST

## Links to affected code

[https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/pools/Pools.sol#L222-L224](https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/pools/Pools.sol#L222-L224)

## Impact

It can leave a deposit amount smaller than DUST.

## Proof of Concept

It does not check whether the remaining `_userDeposits` after withdrawal is more than DUST. You should avoid depositing less than DUST because you won't be able to get it out later unless you deposit it.

```jsx
function withdraw( IERC20 token, uint256 amount ) external nonReentrant
{
    require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
    require( amount > PoolUtils.DUST, "Withdraw amount too small");

@>  _userDeposits[msg.sender][token] -= amount;

    // Send the token to the user
    token.safeTransfer( msg.sender, amount );

    emit TokenWithdrawal(msg.sender, token, amount);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

```diff
function withdraw( IERC20 token, uint256 amount ) external nonReentrant
{
    require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
    require( amount > PoolUtils.DUST, "Withdraw amount too small");

    _userDeposits[msg.sender][token] -= amount;
+   require(_userDeposits[msg.sender][token] > PoolUtils.DUST || _userDeposits[msg.sender][token] == 0, "remain amount too small");
    // Send the token to the user
    token.safeTransfer( msg.sender, amount );

    emit TokenWithdrawal(msg.sender, token, amount);
}
```