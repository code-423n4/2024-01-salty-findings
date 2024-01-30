# [L-01] - It is possible to call `borrowUSDS` with a very low amount

## Vulnerability details
Nothing prevents a user to borrow a very low amount of USDS (for example 1 wei)
The issue is that everytime a user borrow some USDS, an element is added to `_walletsWithBorrowedUSDS`

```soldiity
    function borrowUSDS( uint256 amountBorrowed ) external nonReentrant
		{
		require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
		require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
		require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );

		// Increase the borrowed amount for the user
		usdsBorrowedByUsers[msg.sender] += amountBorrowed;

		// Remember that the user has borrowed USDS (so they can later be checked for sufficient collateralization ratios and liquidated if necessary)
		_walletsWithBorrowedUSDS.add(msg.sender);

		// Mint USDS and send it to the user
		usds.mintTo( msg.sender, amountBorrowed );

		emit BorrowedUSDS(msg.sender, amountBorrowed);
		}
```

This can cause an issue for liquidator who want to find liquidable users, as to high value `numberOfUsersWithBorrowedUSDS` will make the gas cost of the loop to high, possibly causing a revert

```solidity
File: src\stable\CollateralAndLiquidity.sol
345: 	function findLiquidatableUsers() external view returns (address[] memory)
346: 		{
347: 		if ( numberOfUsersWithBorrowedUSDS() == 0 )
348: 			return new address[](0);
349: 
350: 		return findLiquidatableUsers( 0, numberOfUsersWithBorrowedUSDS() - 1 );
351: 		}
352: 	}
```


```solidity
File: src\stable\CollateralAndLiquidity.sol311: 	function findLiquidatableUsers( uint256 startIndex, uint256 endIndex ) public view returns (address[] memory) //..?@audit compare with canUserBeLiquidated
312: 		{
313: 		address[] memory liquidatableUsers = new address[](endIndex - startIndex + 1);
314: 		uint256 count = 0;
315: 
316: 		// Cache
317: 		uint256 totalCollateralShares = totalShares[collateralPoolID];
318: 		uint256 totalCollateralValue = totalCollateralValueInUSD();
319: 
320: 		if ( totalCollateralValue != 0 )
321: 			for ( uint256 i = startIndex; i <= endIndex; i++ )
```

## Impact
The `findLiquidatableUsers` will be unusable.
Liquidator will have issue to search through liquidable users.

## Tools Used
Manual review

## Recommended Mitigation Steps
Force a minimum amount of USDS borrowed for a user, for example 10 or 100 USDS to make this attack impracticable
