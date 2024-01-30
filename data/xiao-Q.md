###  The parameters of Strings.toHexString are incorrectly formatted:
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L171
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L189
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L217
It is recommended to rewrite it in this form:
```
Strings.toHexString(uint256(uint160(address(token))))
```

