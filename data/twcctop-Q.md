## L1 `findLiquidatableUsers` is  possible to run out of gas when user amount is  large

https://github.com/code-423n4/2024-01-salty/blob/ce719503b43ebf086f5147c0f6efc3c0890bf0f9/src/stable/CollateralAndLiquidity.sol#L345-L352
 
 ```solidity
 	function findLiquidatableUsers() external view returns (address[] memory)
		{
		if ( numberOfUsersWithBorrowedUSDS() == 0 )
			return new address[](0);

		return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
		}
	}
```
    `findLiquidatableUsers` does not have a limit on the number of users, and the number of users is large, which may cause the function to run out of gas.



