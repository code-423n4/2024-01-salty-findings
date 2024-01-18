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

### Instances:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L49
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L59
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Flaunch%2FAirdrop.sol#L78

# [09] Function && Variable Naming Convention

### Description:
The linked variables do not conform to the standard naming convention of Solidity whereby `functions and variable names(local and state) utilize the mixedCase format` unless variables are declared as `constant in which case they utilize the UPPER_CASE_WITH_UNDERSCORES format`. `Private variables and functions should lead with an underscore`.

### Instances:


### Recommendation:
> Consider naming conventions utilized by the linked statements are adjusted to reflect the correct type of declaration according to the Solidity.

