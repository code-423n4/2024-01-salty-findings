## `depositDoubleSwapWithdraw` Allows Swaping For Same Pool

## Impact
The function `depositDoubleSwapWithdraw` allows a user to perform a double swap and a withdrawal in one transaction. However, the function does not check if swapTokenIn and swapTokenMiddle are of the same type. Swapping USDC to USDC for ETH causes a user to swap the same token with itself, which is not a valid or useful operation. Swapping the same token with itself would not change the amount or the value of the token, rather consumes more gas than a direct swap from USDC to ETH.
## Proof of Concept
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L395
## Tools Used
manual
## Recommended Mitigation Steps
Check that the tokens are different
`require(swapTokenIn != swapTokenMiddle, "Cannot swap the same token")`
