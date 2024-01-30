### Lack of contract check in ExchangeConfig::setAccessManager
https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L62
```
function setAccessManager(IAccessManager _accessManager) external onlyOwner {
        require(address(_accessManager) != address(0), "_accessManager cannot be address(0)");
      //@audit  // Missing contract check
        accessManager = _accessManager;

        emit AccessManagerSet(_accessManager);
    }
```