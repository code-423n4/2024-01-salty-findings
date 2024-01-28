## Incosistent against PoolUtils::Dust value in the protocol.
The protocol regard Token reserves less than dust(100) to be treated as if they don't exist at all as mention in the comment [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L11-L13)

ArbitrageSearcj.sol::_bisectionSearch checked if Dust <= 0  to return 0 [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L107)and towards the end of the function check Dust < 0 to return 0 [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L133).

All checks done against dust should stay consistent to what the protocol stated in the comment.
