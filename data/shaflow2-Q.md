# Two keeper call performUpkeep function simultaneously, with the latter wasting a significant amount of gas but failing to obtain the expected profit.

## Related code
github:[244](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L244)
```solidity
    function performUpkeep() public nonReentrant
```
## Proof of Concept
In decentralized Upkeep operations, any user has the right to invoke maintenance for a DEX.
The performUpkeep() function consists of eleven sequentially executed operational steps, and the success or failure of each step does not impact the others.  
  
However, there is a possibility that two maintainers simultaneously initiate maintenance operations, causing the latter's transaction to continue but waste a considerable amount of gas without achieving the expected profit. Additionally, there is the potential for malicious transaction front-running: a malicious actor monitors the pending transaction pool, sacrificing a portion of profits and using a higher gas price to prioritize packing their own transaction. In such a scenario, the malicious actor gains a small fraction of the maintainer's expected profit, while causing the original maintainer to waste a significant amount of gas without obtaining the expected returns.  

## Recommended Mitigation Steps
To prevent this situation, it is recommended to set an expected minimum profit, passed as a parameter in performUpkeep(), and check in step2() if the actual profit is greater than performUpKeep(). If the actual profit is less than expected, a revert can be implemented in performUpkeep() to reduce gas wastage.This setup does not hinder the altruistic maintenance initiated. This is especially crucial considering the high gas prices on the Ethereum mainnet and significant gas consumption.  