# Gas Optimization

1) An array's length should be cached to save gas in for-loops.
++i costs less gas compared to i++, about 5 gas per iteration.

uint arrLength = array.length; 
for(uint i; arrLength;) {

    unchecked { ++i; }
}

2) when using block.timestamp use uint64 instead of uint256.
and change deadLine parameter from uint256 into uint64.
  - pools/Pools.sol
  - stable/CollateralAndLiquidity.sol
  - staking/Liquidity.sol


3) If the constructor is not payable, additional opcodes are generated to check if any ETH has been sent in the call


