# 1)  
## Incosistent against PoolUtils::Dust value in the protocol.
The protocol regard Token reserves less than dust(100) to be treated as if they don't exist at all as mention in the comment [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L11-L13)

ArbitrageSearcj.sol::_bisectionSearch checked if Dust <= 0  to return 0 [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L107)and towards the end of the function check Dust < 0 to return 0 [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L133).

All checks done against dust should stay consistent to what the protocol stated in the comment.



# 2)

## Tie in the Highest Number of Votes for pending whistlelisted Tokens.

code reference: 1) https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L431
    2) https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L251

## impact:
In the current implementation, when there is a tie in the highest number of votes for pending whitelisted tokens, the contract only considers the first highest figure it encounters. This behavior may lead to a denial-of-service (DOS) scenario for the second BallotID that is tying with the first one.


This issue could result in a situation where a token tied for the highest number of votes is not taken into consideration for whitelisting. Consequently, the token cannot be finalized until it receives more votes or until the first token with the highest yes votes gets whitelisted.

## Recommendation
To address this issue and improve the fairness of the voting mechanism, I recommend implementing a tie-breaker mechanism that considers all tokens with the highest number of votes.
