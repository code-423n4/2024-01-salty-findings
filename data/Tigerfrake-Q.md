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

# [02] Avoid Block `Timestamp` Manipulation

### Description:
`Block timestamps` have been used historically for a number of purposes, including `entropy for random numbers locking funds for a set amount of time, and different state-changing, time-dependent conditional statements`. 
Because validators have the capacity to slightly alter `timestamps`, using `block timestamps` wrongly in smart contracts can be quite risky.
The time difference between events can be estimated using `block.number` and the average `block time`. However, because `block times` can change and break functionality, it's best to avoid its use.

### Instances:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FProposals.sol#L392
```Solidity
        if (block.timestamp < ballot.ballotMinimumEndTime )
```

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

# [06] For same condition checks, use modifiers

### Description:
The main advantage of using `modifiers` for the same condition checks in different functions is code `reusability and readability`. 
Instead of repeating the same condition check in every function that requires it, you can define a `modifier` once and then apply it to any function that needs it. This makes your code cleaner and easier to maintain.

### Instances:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FProposals.sol#L224
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FProposals.sol#L233

# [07] Line Length

### Description
Max line length must be no more than `120` but many lines are extended past this length.

### Instance:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L11

###Recommendation
> Consider cutting down the line length below 120.

# [08] Incorrect syntax in require() statement 

### Description:
The correct syntax for using `require` in Solidity is `require(!success)`. The space between `!` and `success` is not needed and will cause a compilation error.
```Solidity
            require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );
```

### Instances:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L49
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L59
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L78

# [09] Unbounded loop

### Description:
New items are `pushed` into the following arrays but there is no option to `pop` them out. Currently, the array can grow indefinitely. 
E.g. there’s no maximum limit and there’s no functionality to remove array values.

If the array grows too large, calling relevant functions might run out of gas and revert. 
Calling these functions could result in a DOS condition.
```Solidity
                _userUnstakeIDs[msg.sender].push( unstakeID );
```

### Instances:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fstaking%2FStaking.sol#L71

# [10] Precision Loss When Dividing Odd Integers by Two

### Description:
The contract has a flaw where it may `lose precision when dividing odd integers by two`. This
is because in Solidity, integer division is floor division, meaning that the result of the division operation will be the largest integer less than or equal to the exact result. Therefore,
when an odd integer is divided by two, the result will be rounded down, leading to a loss of precision.

Instances:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fprice_feed%2FPriceAggregator.sol#L139
```Solidity
                uint256 averagePrice = ( priceA + priceB ) / 2;
```

### Recommendation:
When dividing an amount by two, consider taking the first amount as the division result by
two, and the second one to be the total amount minus the first one.

# [11] Integer casting overflow

### Description:
`Solidity >0.8` handles the integer `overflow/underflow` during arithmetic operations. But in case of `casting`, the transaction doesn't `revert` and value overflows.

### Exploiting scenario
In the `claimAllRewards()` function, there's potential issue, but the probability of exploiting is very low when token uses standard decimals and proper total supply. 
However I have to point out this as a warning.

On line 159 if the `(uint256 pendingRewards)` would be higher than `uint128.max`, then the result can overflow and cause mismatch. 

https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fstaking%2FStakingRewards.sol#L159

```Solidity
                        userInfo[poolID].virtualRewards += uint128(pendingRewards);
```
And then in line 161, `claimableRewards` is updated with the original `uint256 pendingRewards` which could be higher than the overflow result above.

 https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fstaking%2FStakingRewards.sol#L161
```Solidity
claimableRewards += pendingRewards;
```

### Recommendation
Be aware of `integer casting overflow` and if this edge case is possible, add the `require statement` to avoid it definitely.
