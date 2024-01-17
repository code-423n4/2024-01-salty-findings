# [G-01] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient. 

### Description:
`SSTORE` from 0 to 1 (or any non-zero value) costs 20000 gas. `SSTORE` from 1 to 2 (or any other non-zero value) costs 5000 gas.
By storing the original value once again, a refund is triggered ( https://eips.ethereum.org/EIPS/eip-2200).
Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.
Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.
See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33

### Instances:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAO.sol#L187
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAO.sol#L194

# [G-02] Make some variables smaller for storage packing and favorizing the user

### Description:
If the following variables can’t be made constant, they should still be made smaller for storage packing.

### Instances:

- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAOConfig.sol#L28
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAOConfig.sol#L57
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FDAOConfig.sol#L61

# [G-3] Use `constants` instead of `type(uintX).max`

### Description:
Using `constants` instead of `type(uintX).max` saves gas in Solidity. This is because the `type(uintX).max` function has to dynamically calculate the maximum value of a `uint256`, which can be expensive in terms of gas. `Constants`, on the other hand, are stored in the `bytecode` of your contract, so they do not have to be recalculated every time you need them. 
Saves 13 GAS

### Instance:
https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FProposals.sol#L165