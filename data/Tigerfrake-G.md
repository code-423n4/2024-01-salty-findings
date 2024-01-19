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

# [G-02] Emit local variables instead of state variable. 
(Save ~100 Gas)

### Instance:
- https://github.com/code-423n4/2024-01-salty/blob/main/src%2FManagedWallet.sol#L42-L55

```Solidity
        function proposeWallets( address _proposedMainWallet, address _proposedConfirmationWallet ) external
                
    //... require statements 
              proposedMainWallet = _proposedMainWallet;
                proposedConfirmationWallet = _proposedConfirmationWallet;

                emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
                }
```

### Description:
Since we are setting our `state variables` `proposedMainWallet` and `proposedConfirmationWallet` to the function parameter `_proposedMainWallet` and `_proposedConfirmationWallet` respectively, we should `emit the function parameter` as it is cheaper to read compared to the `state variable`:

```Solidity
                emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
```

