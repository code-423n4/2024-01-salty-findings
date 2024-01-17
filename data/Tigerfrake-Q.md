# [01] Improper Indentation:

### Description:
The indentation in the following instances is inconsistent, making it harder to read and understand the code.
Example:

```Solidity
                if ( address(swapTokenIn) == address(wbtc))
                if ( address(swapTokenOut) == address(weth))
                        return (wbtc, salt);
```
Proper indentation improves code readability.

### Instances:

- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Farbitrage%2FArbitrageSearch.sol#L35-L37
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Farbitrage%2FArbitrageSearch.sol#L41-L43

### Recommendation:
> Nest the two if checks with proper indentation or use the && operand in one if-check if both checks are required to be valid for these operations

# [02] Avoid Block Timestamp Manipulation
Block timestamps have been used historically for a number of purposes, including entropy for random numbers locking funds for a set amount of time, and different state-changing, time-dependent conditional statements. 
Because validators have the capacity to slightly alter timestamps, using block timestamps wrong in smart contracts can be quite risky.
The time difference between events can be estimated using block.number and the average block time. However, because block times can change and break functionality, it's best to avoid its use.

### Instances:


# [03] Missing or Incomplete NatSpec

### Description
Some functions are missing `@notice/@dev` NatSpec comments for the function, `@param` for all/some of their parameters and `@return` for return values. Given that NatSpec is an important part of code documentation, this affects code comprehension, auditability and usability.

### Instances:
All contracts

### Recommendation
> Consider adding in full NatSpec comments for all functions to have complete code documentation for future use.

# [04] Unchecked External Calls

### Description:
External fucntion calls can introduce vulnerabilities when not properly validated. Trusting external calls without proper checks can lead to unintended behavior or even attacks.

Example: the `withdrawArbitrageProfits()` function fully trusts that `pools.withdraw()` function it calls within it will function as intended without any checks for its success.

### Instances:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAO.sol#L304

# [05] Undocumented Value Literal

### Description:
The value literal 200000 ether is hardcoded to the variable `bootstrappingRewards`, however, it is undocumented and unclearly depicted.

```Solidity
uint256 public bootstrappimgRewards = 200000 ether;
```

### Instance:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAOConfig.sol#L24

### Recommendation:
We advise the special underscore (_) separator to be applied to it (i.e. 200000 would become 200_000) 
