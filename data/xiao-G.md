
## Use bytes(newWebsiteURL).length > 0 to determine whether the string is empty, no need to use keccak256
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251

### Comparison test gas optimization:
#### 1. Use bytes(newWebsiteURL).length✅
```solidity
require(bytes(newWebsiteURL).length > 0,"newWebsiteURL cannot be empty");
```
```
Running 2 tests for src/dao/tests/Proposals.t.sol:TestProposals
[PASS] testProposeWebsiteUpdate() (gas: 477799)
[PASS] testProposeWebsiteUpdateWithEmptyURL() (gas: 127266)
Test result: ok. 2 passed; 0 failed; 0 skipped; finished in 47.70s
```
#### 2. Use keccak256
```solidity
Running 2 tests for src/dao/tests/Proposals.t.sol:TestProposals
[PASS] testProposeWebsiteUpdate() (gas: 478056)
[PASS] testProposeWebsiteUpdateWithEmptyURL() (gas: 127511)
Test result: ok. 2 passed; 0 failed; 0 skipped; finished in 52.79s

Ran 1 test suites: 2 tests passed, 0 failed, 0 skipped (2 total tests)
```

Save gas(257+245)/2=260

