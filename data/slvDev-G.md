## Gas Findings

Initial gas report was saved with `forge snapshot`.

Total gas saved per issue represent `Overall gas change` from `forge snapshot --diff`.

More details from `forge snapshot --diff` can be found in Gas Findings Details.

|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | `x.a = x.a + b` always cheaper than `x.a += b` for Structs | 11 | 778253 |
| [G-02] | Consider using `delete` rather than assigning `false` to clear values | 3 | 21620 |
| [G-03] | Stack variable is only used once | 83 | 262798 |
| [G-04] | Optimize Storage with Byte Truncation for Time Related State Variables | 5 | 100000 |
| [G-05] | Assigning `state` variables directly with struct constructors wastes gas | 4 | 79517 |
| [G-06] | Using `delete` on a `uint/int` variable is cheaper than assigning it to `0` | 7 | 9428 |
| [G-07] | Assembly: Check `msg.sender` using xor and the scratch space | 32 | 672 |
| [G-08] | Use `revert()` to gain maximum gas savings | 139 | 6950 |
| [G-09] | Using `constants` instead of `enum` can save gas | 1 | 0 |
| [G-10] | Optimize Increment and Decrement in loops with `unchecked` keyword | 21 | 1260 |
| [G-11] | Reduce deployment costs by tweaking contracts' metadata | 35 | 371000 |
| [G-12] | Consider using solady's `FixedPointMathLib` | 200 | - |
| [G-13] | Consider using OZ EnumerateSet in place of nested mappings | 5 | 5000 |
| [G-14] | Optimize `require/revert` Statements with Assembly | 139 | 41700 |

## Gas Findings Details

### [G-01] `x.a = x.a + b` always cheaper than `x.a += b` for Structs

Examples:
```solidity
    // direct write to storage   -> 5337 gas
    // struct storage pointer    -> 5341 gas
    // memory struct             -> 4628 gas
    // memory function param     -> 1109 gas
    x.a += b;

    // direct write to storage   -> 5330 gas
    // struct storage pointer    -> 5334 gas
    // memory struct             -> 4625 gas
    // memory function param     -> 1106 gas
    x.a = struct.a + b;
```

Result: 
```bash
	testFinalizeDecreaseParameterBallots() (gas: -45 (-0.000%)) 
	testFinalizeNoChangeParameterBallots() (gas: -46 (-0.001%)) 
	testFinalizeIncreaseParameterBallots() (gas: -45 (-0.001%)) 
	testUnwhitelistTokenDenied() (gas: -57 (-0.003%)) 
	testUnwhitelistTokenApproved() (gas: -57 (-0.003%)) 
	testSuccessfulRewardDistributionAfterWhitelisting() (gas: -57 (-0.003%)) 
	testWhitelistTokenApproved() (gas: -57 (-0.003%)) 
	testProposeTokenUnwhitelistingChangesStateOnlyIfWhitelisted() (gas: -57 (-0.003%)) 
	testUserProposalAfterFinalization() (gas: -57 (-0.004%)) 
	testActiveProposalRestriction() (gas: -57 (-0.004%)) 
	testWhitelistTokenDenied() (gas: -57 (-0.004%)) 
	testPerformUpkeep() (gas: -36 (-0.005%)) 
	testIncludeCountryDenied() (gas: -45 (-0.005%)) 
	testIncludeCountryApproved() (gas: -46 (-0.005%)) 
	testSetWebsiteDenied2() (gas: -45 (-0.006%)) 
	testSetWebsiteApproved() (gas: -57 (-0.007%)) 
	testProposeTokenWhitelisting() (gas: -210 (-0.008%)) 
	testOpenBallotsForTokenWhitelisting() (gas: -210 (-0.008%)) 
	testCallContractApproved() (gas: -57 (-0.009%)) 
	testCallContractDenied() (gas: -57 (-0.010%)) 
	testFinalizedBallotsStillVisible() (gas: -45 (-0.011%)) 
	testSendSaltApproved() (gas: -57 (-0.011%)) 
	testProposeParameterBallotWithInvalidParameterType() (gas: -57 (-0.011%)) 
	testBallotIsApproved() (gas: -46 (-0.011%)) 
	testProposeCallContract() (gas: -57 (-0.011%)) 
	testSendSaltDenied() (gas: -57 (-0.011%)) 
	testProposeTokenWhitelistingMaxPending() (gas: -516 (-0.011%)) 
	testUserCannotVoteOnClosedBallot() (gas: -46 (-0.011%)) 
	testExcludeCountryApproved() (gas: -57 (-0.012%)) 
	testProposeWebsiteUpdate() (gas: -57 (-0.012%)) 
	testTokenWhitelistingBallots() (gas: -363 (-0.012%)) 
	testProposeTokenUnwhitelisting() (gas: -210 (-0.012%)) 
	testBallotForID() (gas: -45 (-0.012%)) 
	testUpdateLastUpkeepTime() (gas: -180 (-0.012%)) 
	testSetWebsiteDenied1() (gas: -57 (-0.013%)) 
	testExcludeCountryDenied() (gas: -57 (-0.013%)) 
	testMarkBallotAsFinalized() (gas: -46 (-0.013%)) 
	testUserHasActiveProposalFlagResetAfterBallotFinalized() (gas: -45 (-0.013%)) 
	testIncorrectApprovalVote() (gas: -57 (-0.013%)) 
	testMarkBallotAsFinalizedRemovesFromOpenBallots() (gas: -45 (-0.013%)) 
	testProposeParameterBallot() (gas: -57 (-0.013%)) 
	testIncorrectParameterVote() (gas: -57 (-0.013%)) 
	testSetContractApproved() (gas: -413 (-0.013%)) 
	testOpenBallots() (gas: -363 (-0.013%)) 
	testSetContractDenied2() (gas: -657 (-0.014%)) 
	testSwapDustRestriction() (gas: -325 (-0.016%)) 
	testSwapReserveBelowDust() (gas: -325 (-0.016%)) 
	testAddLiquidityWithReversedTokenOrder() (gas: -325 (-0.016%)) 
	testPlaceSmallInternalSwap() (gas: -382 (-0.017%)) 
	testPlaceInternalSwap() (gas: -382 (-0.017%)) 
	testRejectDualZapForNonWhitelistedPools() (gas: -325 (-0.019%)) 
	testAddLiquidityToNonWhitelistedPool() (gas: -325 (-0.019%)) 
	testAddLiquidityToBadPair() (gas: -325 (-0.019%)) 
	testCheckSwapK() (gas: -2600 (-0.020%)) 
	testCheckReserves() (gas: -2600 (-0.021%)) 
	testWhitelistingAlreadyWhitelisted() (gas: -57 (-0.021%)) 
	testAddLiquidityEstimate1() (gas: -854 (-0.023%)) 
	testAddLiquidityEstimate2() (gas: -854 (-0.023%)) 
	testUserLiquidationWithDivergentPrices() (gas: -382 (-0.023%)) 
	testInvalidInputs() (gas: -325 (-0.024%)) 
	testProposeDuplicateWebsiteUpdate() (gas: -210 (-0.025%)) 
	testUsersVotingPowerRemovedOnFullUnstake() (gas: -376 (-0.026%)) 
	testSetContractDenied1() (gas: -780 (-0.027%)) 
	testProposeCountryInclusionExclusion() (gas: -210 (-0.028%)) 
	testProposeSameBallotNameMultipleTimesAndForOpenBallot() (gas: -210 (-0.028%)) 
	testIncrementingBallotID() (gas: -210 (-0.029%)) 
	testAddLiquidityReturnValues() (gas: -644 (-0.029%)) 
	testDepositSwapWithdraw() (gas: -650 (-0.029%)) 
	testAddLiquidityRespectsTokenRatioWithNonZeroReserves() (gas: -644 (-0.031%)) 
	testQuorumReachedNotBeforeVotingPower() (gas: -168 (-0.031%)) 
	testFinalizeBallotWithNotFinalizableBallots() (gas: -168 (-0.034%)) 
	testPerformUpkeepMaxGas() (gas: -41238 (-0.037%)) 
	testVoteTallyResetUponChangingVote() (gas: -210 (-0.037%)) 
	testProposeSendSALT() (gas: -210 (-0.038%)) 
	testProposeCountryInclusionExclusionEmptyName() (gas: -57 (-0.041%)) 
	testProposeSetContractAddress() (gas: -210 (-0.042%)) 
	testUnstakeDurationTooLong() (gas: -57 (-0.043%)) 
	testUnstakeDurationTooShort() (gas: -57 (-0.043%)) 
	testProposeParameterBallotBeforeTimestamp() (gas: -57 (-0.043%)) 
	testDuplicateParameterBallot() (gas: -210 (-0.044%)) 
	testDuplicateApprovalBallot() (gas: -210 (-0.044%)) 
	testUserLiquidationWithTwoFailedPriceFeeds() (gas: -382 (-0.044%)) 
	testProposeSetContractAddressRejectsZeroAddress() (gas: -57 (-0.044%)) 
	testProposeCallContractZeroAddressRejection() (gas: -57 (-0.044%)) 
	testProposeWebsiteUpdateWithEmptyURL() (gas: -57 (-0.045%)) 
	testParameterBallotVoting() (gas: -363 (-0.046%)) 
	testUnstakeMoreThanStaked() (gas: -57 (-0.047%)) 
	testCollateralDepositersCanReceiveStakingRewards() (gas: -400 (-0.047%)) 
	testUserRewardsForPools() (gas: -114 (-0.048%)) 
	testAddLiquidityWithFlippedTokens() (gas: -1708 (-0.048%)) 
	testDepositDoubleSwapWithdraw() (gas: -1910 (-0.050%)) 
	testClaimRewardsWithNoShares() (gas: -111 (-0.050%)) 
	testMaximumLiquidityRemoval() (gas: -1055 (-0.050%)) 
	testRewardsCannotBeClaimedWithoutShares() (gas: -93 (-0.051%)) 
	testUserBorrowingWithBadPriceFeed() (gas: -382 (-0.055%)) 
	testClaimAllRewards() (gas: -150 (-0.055%)) 
	testDualZapInLiquidity() (gas: -1326 (-0.056%)) 
	testClaimAllRewardsMultipleValidPools() (gas: -150 (-0.058%)) 
	testAddLiquidity() (gas: -1300 (-0.059%)) 
	testCastVote() (gas: -363 (-0.060%)) 
	testClaimAirdrop() (gas: -433 (-0.061%)) 
	testStakingMoreThanBalance() (gas: -57 (-0.061%)) 
	testRepaymentAdjustsAccountingCorrectly() (gas: -305 (-0.061%)) 
	testStakeExcessSALT() (gas: -57 (-0.061%)) 
	testMaxWithdrawableCollateralAfterRepayment() (gas: -306 (-0.061%)) 
	testMultipleSwaps() (gas: -8990 (-0.062%)) 
	testStakeNegativeAmount() (gas: -57 (-0.062%)) 
	testRepayUSDS() (gas: -306 (-0.062%)) 
	testLiquidateUserWithoutBorrowedUSDS() (gas: -306 (-0.062%)) 
	testUserCannotVoteAfterUnstakingAllSALT() (gas: -376 (-0.062%)) 
	testBorrowBalanceNotNegativeAfterRepayment() (gas: -306 (-0.062%)) 
	testStakeWithNoAllowance() (gas: -57 (-0.063%)) 
	testClaimAirdrop2() (gas: -433 (-0.064%)) 
	testClaimedMappingAfterAirdropClaim() (gas: -433 (-0.064%)) 
	testStep9() (gas: -36 (-0.064%)) 
	testSeriesOfTrades() (gas: -1910 (-0.065%)) 
	testWinningParameterVote() (gas: -516 (-0.066%)) 
	testZapping() (gas: -25344 (-0.066%)) 
	testDepositAndWithdrawLiquidity2() (gas: -1584 (-0.067%)) 
	testProcessRewardsFromPOL() (gas: -800 (-0.067%)) 
	testDepositAndWithdrawMaximumLiquidity() (gas: -1584 (-0.068%)) 
	testZappingDust() (gas: -7920 (-0.068%)) 
	testArbitrageFailed() (gas: -1528 (-0.070%)) 
	testMultipleInteractions() (gas: -2056 (-0.070%)) 
	testComprehensivePerformUpkeep() (gas: -2168 (-0.071%)) 
	testPerformUpkeepZeroPrice() (gas: -2168 (-0.071%)) 
	testDoublePerformUpkeep() (gas: -2952 (-0.071%)) 
	testReserveAndBalanceManagement() (gas: -319 (-0.071%)) 
	testCanUserBeLiquidatedUnderThreshold() (gas: -382 (-0.073%)) 
	testProposeWithInsufficientStake() (gas: -363 (-0.073%)) 
	testTotalVotesCastForBallot() (gas: -516 (-0.074%)) 
	testMaxWithdrawableLP_and_maxBorrowableUSDS() (gas: -382 (-0.074%)) 
	testUserCollateralValueInUSD() (gas: -382 (-0.076%)) 
	testSuccessStep7() (gas: -917 (-0.079%)) 
	testCanUserBeLiquidated_TrueIfWalletOnBorrowedListAndUndercollateralized() (gas: -382 (-0.080%)) 
	testChangeVotesAfterUnstakingSALT() (gas: -695 (-0.080%)) 
	testWithdrawnPoolAddSALTRewards() (gas: -382 (-0.080%)) 
	testArbitrage4() (gas: -1910 (-0.080%)) 
	testArbitrage1() (gas: -1910 (-0.080%)) 
	testArbitrage2() (gas: -1910 (-0.080%)) 
	testArbitrage3() (gas: -1910 (-0.080%)) 
	testArbitrageReservesMin() (gas: -1910 (-0.080%)) 
	testRevertStep2() (gas: -36805 (-0.081%)) 
	testRevertStep3() (gas: -37187 (-0.081%)) 
	testRevertStep4() (gas: -37187 (-0.081%)) 
	testRevertStep9() (gas: -37533 (-0.082%)) 
	testRevertStep6() (gas: -37569 (-0.082%)) 
	testRevertStep1() (gas: -37569 (-0.082%)) 
	testRevertStep10() (gas: -37569 (-0.082%)) 
	testRevertStep5() (gas: -37569 (-0.082%)) 
	testRevertStep11() (gas: -37569 (-0.082%)) 
	testRevertStep7() (gas: -37569 (-0.082%)) 
	testUserCollateralValueInUSD2() (gas: -382 (-0.082%)) 
	testRevertStep8() (gas: -37569 (-0.082%)) 
	testApprovalBallotVoting() (gas: -835 (-0.083%)) 
	testClaimRewardsFromInvalidPools() (gas: -18 (-0.084%)) 
	testBorrowMaxPlusOneUSDS() (gas: -382 (-0.084%)) 
	testUserLiquidationWithTwoGoodFeeds() (gas: -1584 (-0.086%)) 
	testRepayUSDSExceedBorrowed() (gas: -382 (-0.087%)) 
	testSuccessStep9() (gas: -640 (-0.090%)) 
	testComprehensive() (gas: -2955 (-0.090%)) 
	testSuccessStep1() (gas: -917 (-0.090%)) 
	testNoDuplicateWalletsInBorrowUSDS() (gas: -683 (-0.091%)) 
	testMaxBorrowableUSDS() (gas: -1060 (-0.095%)) 
	testPerformUpkeepMaxGas() (gas: -5486 (-0.095%)) 
	testSuccessStep5() (gas: -1528 (-0.095%)) 
	testSuccessfulBalanceIncreaseAfterPerformUpkeep() (gas: -917 (-0.095%)) 
	testSuccessfulBalanceIncreaseAfterPerformUpkeepWithLimit() (gas: -917 (-0.095%)) 
	testPerformUpkeepSwapsCorrectTokenAmounts() (gas: -917 (-0.095%)) 
	testCannotBorrowMoreThanMaxBorrowableLimit() (gas: -306 (-0.096%)) 
	testSuccessStep3() (gas: -1223 (-0.096%)) 
	testCanFinalizeBallot() (gas: -892 (-0.097%)) 
	testUserPositionFunctions() (gas: -683 (-0.098%)) 
	testSuccessStep4() (gas: -1528 (-0.098%)) 
	testLiquidatePositionWithZeroBorrowedAmount() (gas: -306 (-0.101%)) 
	testFormPOL() (gas: -306 (-0.101%)) 
	testCannotClaimRewardsFromZeroPendingRewards() (gas: -150 (-0.101%)) 
	testArbitrage5() (gas: -4472 (-0.103%)) 
	testCorrectPOLFormationAndDistribution() (gas: -306 (-0.103%)) 
	testUserPendingReward() (gas: -210 (-0.105%)) 
	testPerformUpkeep() (gas: -1061 (-0.105%)) 
	testSaltyFeed() (gas: -611 (-0.106%)) 
	testRequiredQuorumForBallotType() (gas: -210 (-0.107%)) 
	testAddLiquidityAndIncreaseShare() (gas: -306 (-0.108%)) 
	testTotalCollateralValueInUSD() (gas: -1061 (-0.109%)) 
	testFindLiquidatablePositions_noLiquidatablePositions() (gas: -1061 (-0.109%)) 
	testWithdrawingMoreThanDeposited() (gas: -305 (-0.110%)) 
	testExpeditedUnstakeBurnsSalt() (gas: -301 (-0.111%)) 
	testDepositCollateralAndIncreaseTotalShares() (gas: -305 (-0.112%)) 
	testRecoverSALTAfterUnstaking() (gas: -300 (-0.112%)) 
	testLargeUSDSReservesETH() (gas: -305 (-0.112%)) 
	testCannotWithdrawPastDeadline() (gas: -306 (-0.112%)) 
	testLargeUSDSReservesBTC() (gas: -306 (-0.112%)) 
	testDepositCollateralAndIncreaseShare() (gas: -382 (-0.116%)) 
	testMinimumCollateralValueForBorrowing() (gas: -684 (-0.119%)) 
	testGetPriceConsistency() (gas: -611 (-0.125%)) 
	testRecoverSALTFromUnstakeNotBelongingToSender() (gas: -376 (-0.125%)) 
	testCorrectOperationOfGetPriceBTCAndETHWithSufficientReserves() (gas: -612 (-0.126%)) 
	testCancelUnstakeNonExistent() (gas: -376 (-0.129%)) 
	testRemoveLiquidity() (gas: -55820 (-0.130%)) 
	testUserCooldowns() (gas: -267 (-0.131%)) 
	testUnstakingDecreasesOnlyByUnstakedAmount() (gas: -376 (-0.131%)) 
	testFindLiquidatablePositions() (gas: -1645 (-0.131%)) 
	testUnstake() (gas: -376 (-0.132%)) 
	testNumberOfOpenPositions() (gas: -1645 (-0.136%)) 
	testExcessTokensAreReverted() (gas: -683 (-0.143%)) 
	testMaxRewardValueForCallingLiquidation() (gas: -1267 (-0.144%)) 
	testCancelUnstake2() (gas: -433 (-0.144%)) 
	testUserShareForPool_multipleCollateralPositions() (gas: -1438 (-0.145%)) 
	testUserShareForPools() (gas: -420 (-0.145%)) 
	testRewardValueForCallingLiquidation() (gas: -1267 (-0.150%)) 
	testRecoverSALTFromNonPendingUnstake() (gas: -433 (-0.150%)) 
	testUnstakingSALTWithVariousDurations() (gas: -1014 (-0.155%)) 
	testCancelCompletedUnstakeRevert() (gas: -376 (-0.156%)) 
	testLiquidateUserMarginallyAboveThreshold() (gas: -1267 (-0.158%)) 
	testLiquidatePositionFailure() (gas: -1267 (-0.158%)) 
	testTotalSharesForPool() (gas: -890 (-0.159%)) 
	testWithdrawCollateralAndSharesUpdate() (gas: -890 (-0.160%)) 
	testBorrowerPositionBeforeAndAfterRemovingLiquidity() (gas: -889 (-0.160%)) 
	testBorrowUSDSAndLiquidation() (gas: -1267 (-0.160%)) 
	testMultipleSharesAndRewards() (gas: -417 (-0.160%)) 
	testLiquidationRewardNotExceedingLimits() (gas: -1267 (-0.161%)) 
	testStakeSALTWithZeroBalance() (gas: -210 (-0.162%)) 
	testRecoverSalt() (gas: -1320 (-0.162%)) 
	testSimultaneousStaking() (gas: -363 (-0.165%)) 
	testMinLiquidityReceived() (gas: -325 (-0.166%)) 
	testLiquidateSelf() (gas: -1267 (-0.167%)) 
	testLiquidatePosition() (gas: -1645 (-0.170%)) 
	testMultipleUnstakeRecovery() (gas: -1014 (-0.173%)) 
	testMultiplePendingUnstakesWithDifferentCompletionTimes() (gas: -1014 (-0.174%)) 
	testUnstakesForUser() (gas: -1014 (-0.174%)) 
	testUnderwaterPosition() (gas: -2229 (-0.175%)) 
	testUserUnstakeIDs() (gas: -1167 (-0.176%)) 
	testPermissionChecksForCancelUnstakeAndRecoverSALT() (gas: -848 (-0.178%)) 
	testUnstakeStateUpdatesAfterCancelAndRecover() (gas: -848 (-0.180%)) 
	testCannotRecoverSALTFromCancelledUnstake() (gas: -529 (-0.181%)) 
	testPendingRewardsForPools() (gas: -2122 (-0.182%)) 
	testLiquidateUserUpdatesUsersWithBorrowedUSDS() (gas: -2229 (-0.183%)) 
	testCancelUnstake() (gas: -848 (-0.183%)) 
	testUserBalanceXSALT2() (gas: -1320 (-0.183%)) 
	testAliceBobCharlieMultipleSharesAndShareRewards() (gas: -471 (-0.184%)) 
	testUserLiquidationTwice() (gas: -2228 (-0.184%)) 
	testWithdrawLiquidityAfterPoolUnwhitelisted() (gas: -890 (-0.187%)) 
	testTotalSharesForPools() (gas: -726 (-0.187%)) 
	testUserDepositBorrowDepositAndLiquidate() (gas: -1900 (-0.188%)) 
	testCannotLiquidateWithinCertainTimeFrame() (gas: -1596 (-0.188%)) 
	testUnstakeBeforeCooldown() (gas: -634 (-0.192%)) 
	testUSDSBalanceDecreasedAfterPOLWithdrawal() (gas: -1474 (-0.195%)) 
	testDAOWithdrawPOL() (gas: -889 (-0.199%)) 
	testMaintainCorrectUnstakeIDsListAfterUnstakesAndCancels() (gas: -1486 (-0.199%)) 
	testWithdrawPOLProperlyWithdrawsLiquidity() (gas: -890 (-0.200%)) 
	testUserCollateralValueInUSD_multiplePositions() (gas: -1061 (-0.200%)) 
	testUserBalanceXSALT() (gas: -1473 (-0.203%)) 
	testStakingVariousAmounts() (gas: -516 (-0.213%)) 
	testWithdrawalOfPOLUpdatesDAOsPOLBalance() (gas: -1779 (-0.219%)) 
	testMultiAddRemoveLiquidity() (gas: -7336 (-0.222%)) 
	testAddLiquidityWithoutZapping() (gas: -1316 (-0.226%)) 
	testMultipleUserStakingClaiming() (gas: -3611 (-0.227%)) 
	testDepositAndWithdrawCollateral2() (gas: -1268 (-0.233%)) 
	testMultipleUserStakingClaiming() (gas: -3654 (-0.237%)) 
	testAddingDustLiquidity() (gas: -472 (-0.240%)) 
	testValidWithdrawLiquidityAndClaim() (gas: -890 (-0.253%)) 
	testtransferStakedSaltFromAirdropToUser() (gas: -529 (-0.260%)) 
	testDepositAndWithdrawCollateral() (gas: -1852 (-0.280%)) 
	testIncreaseDecreaseShare() (gas: -376 (-0.317%)) 
	testUserCanIncreaseOrDecreaseShare() (gas: -376 (-0.321%)) 
	testCorrectXSALTAssignment() (gas: -669 (-0.346%)) 
	testAddRemoveLiquidityBeforeExchangeLive() (gas: -11216 (-0.351%)) 
	testSetContracts() (gas: -11216 (-0.352%)) 
	testMultipleUsersActions() (gas: -1073 (-0.364%)) 
	testReturnValues() (gas: -319 (-0.372%)) 
	testAddLiquidityWithExceededMaxAmount0() (gas: -319 (-0.377%)) 
	testLiveBalanceCheck() (gas: -12194 (-0.378%)) 
	testAddLiquidityWithExceededMaxAmount1() (gas: -319 (-0.389%)) 
	testGasAddLiquidity() (gas: -319 (-0.419%)) 
	testSequentialLiquidityAdjustment() (gas: -3606 (-0.445%)) 
	testMinLiquidityAndReclaimedAmounts() (gas: -730 (-1.218%)) 
	testUnimodalHypothesis(uint256,uint256,uint256,uint256,uint256,uint256,uint256) (gas: -28 (-1.772%)) 
	Overall gas change: -778253 (-0.063%)
```

<details>
<summary><i>11 issue instances in 2 files:</i></summary>

```solidity
File: src/pools/Pools.sol

100: reserves.reserve0 += uint128(maxAmount0)
101: reserves.reserve1 += uint128(maxAmount1)
125: reserves.reserve0 += uint128(addedAmount0)
126: reserves.reserve1 += uint128(addedAmount1)
182: reserves.reserve0 -= uint128(reclaimedA)
183: reserves.reserve1 -= uint128(reclaimedB)
```
[100](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L100) | [101](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L101) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L126) | [182](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L182) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L183)

```solidity
File: src/staking/StakingRewards.sol

83: user.virtualRewards += uint128(virtualRewardsToAdd)
88: user.userShare += uint128(increaseShareAmount)
125: user.userShare -= uint128(decreaseShareAmount)
126: user.virtualRewards -= uint128(virtualRewardsToRemove)
159: userInfo[poolID].virtualRewards += uint128(pendingRewards)
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L83) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L88) | [125](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L125) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L126) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L159)
</details>


### [G-02] Consider using `delete` rather than assigning `false` to clear values

Result:
```bash
	testIncludeCountryApproved() (gas: -20 (-0.002%)) 
	testRevertStep6() (gas: -1800 (-0.004%)) 
	testRevertStep1() (gas: -1800 (-0.004%)) 
	testRevertStep10() (gas: -1800 (-0.004%)) 
	testRevertStep5() (gas: -1800 (-0.004%)) 
	testRevertStep9() (gas: -1800 (-0.004%)) 
	testRevertStep11() (gas: -1800 (-0.004%)) 
	testRevertStep7() (gas: -1800 (-0.004%)) 
	testRevertStep3() (gas: -1800 (-0.004%)) 
	testRevertStep4() (gas: -1800 (-0.004%)) 
	testRevertStep8() (gas: -1800 (-0.004%)) 
	testRevertStep2() (gas: -1800 (-0.004%)) 
	testDAOConstructor() (gas: -1800 (-0.039%)) 
	Overall gas change: -21620 (-0.002%)
```

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/dao/DAO.sol

187: excludedCountries[ ballot.string1 ] = false
```
[187](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L187)

```solidity
File: src/dao/Proposals.sol

143: ballot.ballotIsLive = false
147: _userHasActiveProposal[userThatPostedBallot] = false
```
[143](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L143) | [147](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L147)
</details>


### [G-03] Stack variable is only used once

Test only one case due to the large number of instances.

Test case:
```diff
File: src/dao/DAO.sol
-			Ballot memory ballot = proposals.ballotForID(ballotID);
-			_executeApproval( ballot );
+			_executeApproval( proposals.ballotForID(ballotID) );
```

Result: 
```bash
	testSuccessfulRewardDistributionAfterWhitelisting() (gas: -1 (-0.000%)) 
	testWhitelistTokenApproved() (gas: -1 (-0.000%)) 
	testWhitelistTokenDenied() (gas: -1 (-0.000%)) 
	testUnwhitelistTokenDenied() (gas: -2 (-0.000%)) 
	testCallContractDenied() (gas: -1 (-0.000%)) 
	testSendSaltDenied() (gas: -1 (-0.000%)) 
	testSetContractDenied1() (gas: -6 (-0.000%)) 
	testSetWebsiteDenied1() (gas: -1 (-0.000%)) 
	testExcludeCountryDenied() (gas: -1 (-0.000%)) 
	testUnwhitelistTokenApproved() (gas: -26 (-0.001%)) 
	testIncludeCountryDenied() (gas: -20 (-0.002%)) 
	testSetWebsiteDenied2() (gas: -21 (-0.003%)) 
	testSetContractDenied2() (gas: -125 (-0.003%)) 
	testCallContractApproved() (gas: -25 (-0.004%)) 
	testSendSaltApproved() (gas: -25 (-0.005%)) 
	testIncludeCountryApproved() (gas: -40 (-0.005%)) 
	testCoreUniswapFeedConstructor(address,address,address) (gas: -61 (-0.005%)) 
	testSetContractApproved() (gas: -160 (-0.005%)) 
	testExcludeCountryApproved() (gas: -25 (-0.005%)) 
	testSetWebsiteApproved() (gas: -50 (-0.006%)) 
	testRevertStep6() (gas: -21849 (-0.048%)) 
	testRevertStep1() (gas: -21849 (-0.048%)) 
	testRevertStep10() (gas: -21849 (-0.048%)) 
	testRevertStep5() (gas: -21849 (-0.048%)) 
	testRevertStep9() (gas: -21849 (-0.048%)) 
	testRevertStep11() (gas: -21849 (-0.048%)) 
	testRevertStep7() (gas: -21849 (-0.048%)) 
	testRevertStep3() (gas: -21849 (-0.048%)) 
	testRevertStep4() (gas: -21849 (-0.048%)) 
	testRevertStep8() (gas: -21849 (-0.048%)) 
	testRevertStep2() (gas: -21849 (-0.048%)) 
	testDAOConstructor() (gas: -21866 (-0.469%)) 
	Overall gas change: -262798 (-0.021%)
```

<details>
<summary><i>83 issue instances in 22 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

/// @audit - `profitRightOfMidpoint` variable
86: int256 profitRightOfMidpoint = int256(amountOut) - int256(midpoint)
```
[86](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L86)

```solidity
File: src/dao/DAO.sol

/// @audit - `ballot` variable
223: Ballot memory ballot = proposals.ballotForID(ballotID)
/// @audit - `saltBalance` variable
246: uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) )
/// @audit - `bestWhitelistingBallotID` variable
250: uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes()
/// @audit - `pool1` variable
257: bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() )
/// @audit - `pool2` variable
258: bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() )
/// @audit - `remainingSALT` variable
345: uint256 remainingSALT = claimedSALT - amountToSendToTeam
/// @audit - `saltToBurn` variable
348: uint256 saltToBurn = ( remainingSALT * daoConfig.percentPolRewardsBurned() ) / 100
/// @audit - `poolID` variable
364: bytes32 poolID = PoolUtils._poolID(tokenA, tokenB)
/// @audit - `liquidityToWithdraw` variable
369: uint256 liquidityToWithdraw = (liquidityHeld * percentToLiquidate) / 100
```
[223](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L223) | [246](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L246) | [250](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L250) | [257](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L257) | [258](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L258) | [345](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L345) | [348](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L348) | [364](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L364) | [369](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L369)

```solidity
File: src/dao/Proposals.sol

/// @audit - `totalStaked` variable
89: uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT)
/// @audit - `userXSalt` variable
94: uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT )
/// @audit - `ballotMinimumEndTime` variable
105: uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration()
/// @audit - `userThatPostedBallot` variable
146: address userThatPostedBallot = _usersThatProposedBallots[ballotID]
/// @audit - `ballotName` variable
157: string memory ballotName = string.concat("parameter:", Strings.toString(parameterType) )
/// @audit - `ballotName` variable
171: string memory ballotName = string.concat("whitelist:", Strings.toHexString(address(token)) )
/// @audit - `ballotName` variable
189: string memory ballotName = string.concat("unwhitelist:", Strings.toHexString(address(token)) )
/// @audit - `balance` variable
201: uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) )
/// @audit - `maxSendable` variable
202: uint256 maxSendable = balance * 5 / 100
/// @audit - `ballotName` variable
207: string memory ballotName = "sendSALT"
/// @audit - `ballotName` variable
217: string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) )
/// @audit - `ballotName` variable
226: string memory ballotName = string.concat("include:", country )
/// @audit - `ballotName` variable
235: string memory ballotName = string.concat("exclude:", country )
/// @audit - `ballotName` variable
244: string memory ballotName = string.concat("setContract:", contractName )
/// @audit - `ballotName` variable
253: string memory ballotName = string.concat("setURL:", newWebsiteURL )
/// @audit - `totalSupply` variable
334: uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply()
/// @audit - `ballot` variable
346: Ballot memory ballot = ballots[ballotID]
/// @audit - `quorum` variable
419: uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN)
```
[89](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L89) | [94](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L94) | [105](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L105) | [146](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L146) | [157](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L157) | [171](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L171) | [189](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L189) | [201](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L201) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L202) | [207](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L207) | [217](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L217) | [226](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L226) | [235](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L235) | [244](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L244) | [253](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L253) | [334](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L334) | [346](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L346) | [419](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L419)

```solidity
File: src/launch/BootstrapBallot.sol

/// @audit - `messageHash` variable
53: bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender))
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L53)

```solidity
File: src/launch/InitialDistribution.sol

/// @audit - `poolIDs` variable
68: bytes32[] memory poolIDs = poolsConfig.whitelistedPools()
```
[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L68)

```solidity
File: src/pools/PoolsConfig.sol

/// @audit - `poolID1` variable
146: bytes32 poolID1 = PoolUtils._poolID( token, wbtc )
/// @audit - `poolID2` variable
150: bytes32 poolID2 = PoolUtils._poolID( token, weth )
```
[146](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L146) | [150](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L150)

```solidity
File: src/pools/PoolStats.sol

/// @audit - `poolID` variable
40: bytes32 poolID = PoolUtils._poolID( arbToken2, arbToken3 )
/// @audit - `poolID` variable
63: bytes32 poolID = PoolUtils._poolID( tokenA, tokenB )
```
[40](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L40) | [63](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L63)

```solidity
File: src/pools/Pools.sol

/// @audit - `arbitrageProfit` variable
356: uint256 arbitrageProfit = _attemptArbitrage( swapTokenIn, swapTokenOut, swapAmountIn )
/// @audit - `middleAmountOut` variable
399: uint256 middleAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenMiddle, swapAmountIn, 0 )
```
[356](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L356) | [399](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L399)

```solidity
File: src/pools/PoolMath.sol

/// @audit - `C` variable
192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 )
/// @audit - `discriminant` variable
193: uint256 discriminant = B * B + 4 * A * C
```
[192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192) | [193](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L193)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

/// @audit - `answerDelay` variable
43: uint256 answerDelay = block.timestamp - _answerTimestamp
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L43)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

/// @audit - `tick` variable
59: int24 tick = int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)))
```
[59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59)

```solidity
File: src/price_feed/PriceAggregator.sol

/// @audit - `price1` variable
178: uint256 price1 = _getPriceBTC(priceFeed1)
/// @audit - `price2` variable
179: uint256 price2 = _getPriceBTC(priceFeed2)
/// @audit - `price3` variable
180: uint256 price3 = _getPriceBTC(priceFeed3)
/// @audit - `price1` variable
190: uint256 price1 = _getPriceETH(priceFeed1)
/// @audit - `price2` variable
191: uint256 price2 = _getPriceETH(priceFeed2)
/// @audit - `price3` variable
192: uint256 price3 = _getPriceETH(priceFeed3)
```
[178](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L178) | [179](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L179) | [180](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L180) | [190](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L190) | [191](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L191) | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L192)

```solidity
File: src/rewards/Emissions.sol

/// @audit - `saltBalance` variable
52: uint256 saltBalance = salt.balanceOf( address( this ) )
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L52)

```solidity
File: src/rewards/RewardsEmitter.sol

/// @audit - `numeratorMult` variable
112: uint256 numeratorMult = timeSinceLastUpkeep * rewardsConfig.rewardsEmitterDailyPercentTimes1000()
/// @audit - `denominatorMult` variable
113: uint256 denominatorMult = 1 days * 100000
/// @audit - `sum` variable
115: uint256 sum = 0
```
[112](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L112) | [113](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L113) | [115](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L115)

```solidity
File: src/rewards/SaltRewards.sol

/// @audit - `amountPerPool` variable
84: uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length
/// @audit - `liquidityRewardsAmount` variable
144: uint256 liquidityRewardsAmount = remainingRewards - stakingRewardsAmount
```
[84](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L84) | [144](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L144)

```solidity
File: src/stable/StableConfig.sol

/// @audit - `remainingRatioAfterReward` variable
52: uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1
/// @audit - `remainingRatioAfterReward` variable
128: uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L52) | [128](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L128)

```solidity
File: src/stable/CollateralAndLiquidity.sol

/// @audit - `btcPrice` variable
198: uint256 btcPrice = priceAggregator.getPriceBTC()
/// @audit - `ethPrice` variable
199: uint256 ethPrice = priceAggregator.getPriceETH()
/// @audit - `btcValue` variable
202: uint256 btcValue = ( amountBTC * btcPrice ) / wbtcTenToTheDecimals
/// @audit - `ethValue` variable
203: uint256 ethValue = ( amountETH * ethPrice ) / wethTenToTheDecimals
/// @audit - `userWBTC` variable
232: uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares
/// @audit - `userWETH` variable
233: uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares
/// @audit - `maxWithdrawableValue` variable
258: uint256 maxWithdrawableValue = userCollateralValue - requiredCollateralValueAfterWithdrawal
/// @audit - `userCollateralValue` variable
304: uint256 userCollateralValue = userCollateralValueInUSD(wallet)
/// @audit - `totalCollateralShares` variable
317: uint256 totalCollateralShares = totalShares[collateralPoolID]
/// @audit - `minCollateralValue` variable
326: uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100
/// @audit - `minCollateral` variable
329: uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue
```
[198](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L198) | [199](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L199) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L202) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L203) | [232](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L233) | [258](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L258) | [304](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L304) | [317](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L317) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L329)

```solidity
File: src/staking/StakingRewards.sol

/// @audit - `userInfo` variable
149: mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[msg.sender]
/// @audit - `userInfo` variable
294: mapping(bytes32=>UserShareInfo) storage userInfo = _userShareInfo[wallet]
```
[149](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L149) | [294](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L294)

```solidity
File: src/staking/Staking.sol

/// @audit - `completionTime` variable
65: uint256 completionTime = block.timestamp + numWeeks * ( 1 weeks )
/// @audit - `u` variable
68: Unstake memory u = Unstake( UnstakeState.PENDING, msg.sender, amountUnstaked, claimableSALT, completionTime, unstakeID )
/// @audit - `index` variable
162: uint256 index
/// @audit - `percentAboveMinimum` variable
207: uint256 percentAboveMinimum = 100 - minUnstakePercent
/// @audit - `numerator` variable
210: uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) )
```
[65](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L65) | [68](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L68) | [162](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L162) | [207](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L207) | [210](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L210)

```solidity
File: src/AccessManager.sol

/// @audit - `messageHash` variable
53: bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet))
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L53)

```solidity
File: src/SigningTools.sol

/// @audit - `recoveredAddress` variable
26: address recoveredAddress = ecrecover(messageHash, v, r, s)
```
[26](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L26)

```solidity
File: src/Upkeep.sol

/// @audit - `rewardAmount` variable
119: uint256 rewardAmount = withdrawnAmount * daoConfig.upkeepRewardPercent() / 100
/// @audit - `amountOfWETH` variable
152: uint256 amountOfWETH = wethBalance * stableConfig.percentArbitrageProfitsForStablePOL() / 100
/// @audit - `amountOfWETH` variable
165: uint256 amountOfWETH = wethBalance * daoConfig.arbitrageProfitsPercentPOL() / 100
/// @audit - `amountSALT` variable
178: uint256 amountSALT = pools.depositSwapWithdraw( weth, salt, wethBalance, 0, block.timestamp )
/// @audit - `timeSinceLastUpkeep` variable
186: uint256 timeSinceLastUpkeep = block.timestamp - lastUpkeepTimeEmissions
/// @audit - `profitsForPools` variable
196: uint256[] memory profitsForPools = pools.profitsForWhitelistedPools()
/// @audit - `poolIDs` variable
198: bytes32[] memory poolIDs = poolsConfig.whitelistedPools()
/// @audit - `releaseableAmount` variable
233: uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt))
/// @audit - `daoWETH` variable
287: uint256 daoWETH = pools.depositedUserBalance( address(dao), weth )
```
[119](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L119) | [152](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L152) | [165](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L165) | [178](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L178) | [186](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L186) | [196](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L196) | [198](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L198) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L233) | [287](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L287)
</details>


### [G-04] Optimize Storage with Byte Truncation for Time Related State Variables

Certain state variables, particularly timestamps, can be safely stored using `uint32`. 
By optimizing these variables, contracts can utilize storage more efficiently.
This not only results in a reduction in the initial gas costs (due to fewer Gsset operations) but also provides savings in subsequent read and write operations.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: src/dao/DAOConfig.sol

/// @audit - State variables after packing use 8 storage slots, but can be packed to 5 storage slots.
//	uint256 public upkeepRewardPercent = 5;
//	uint256 public arbitrageProfitsPercentPOL = 20;
//	uint256 public percentPolRewardsBurned = 50;
//	uint256 public bootstrappingRewards = 200000 ether;
//	uint32 public maxPendingTokensForWhitelisting = 5;
//	uint32 public requiredProposalPercentStakeTimes1000 = 500;
//	uint32 public ballotMinimumDuration = 10 days;
//	uint32 public baseBallotQuorumPercentTimes1000 = 10 * 1000;

9: contract DAOConfig is IDAOConfig, Ownable
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L9)

```solidity
File: src/launch/BootstrapBallot.sol

/// @audit - State variables after packing use 4 storage slots, but can be packed to 2 storage slots.
//	mapping(address => bool) public hasVoted;
//	uint32 public startExchangeNo;
//	uint32 public startExchangeYes;
//	bool public startExchangeApproved;
//	bool public ballotFinalized;

13: contract BootstrapBallot is IBootstrapBallot, ReentrancyGuard
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L13)

```solidity
File: src/rewards/RewardsConfig.sol

/// @audit - State variables after packing use 4 storage slots, but can be packed to 3 storage slots.
//	uint256 public percentRewardsSaltUSDS = 10;
//	uint256 public stakingRewardsPercent = 50;
//	uint32 public emissionsWeeklyPercentTimes1000 = 500;
//	uint32 public rewardsEmitterDailyPercentTimes1000 = 1000;

9: contract RewardsConfig is IRewardsConfig, Ownable
```
[9](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L9)

```solidity
File: src/ManagedWallet.sol

/// @audit - State variables after packing use 5 storage slots, but can be packed to 4 storage slots.
//	address public proposedConfirmationWallet;
//	uint32 public activeTimelock;
//	address public proposedMainWallet;
//	address public confirmationWallet;
//	address public mainWallet;

12: contract ManagedWallet is IManagedWallet
```
[12](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L12)

```solidity
File: src/Upkeep.sol

/// @audit - State variables after packing use 2 storage slots, but can be packed to 1 storage slots.
//	uint32 public lastUpkeepTimeRewardsEmitters;
//	uint32 public lastUpkeepTimeEmissions;

39: contract Upkeep is IUpkeep, ReentrancyGuard
```
[39](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L39)
</details>


### [G-05] Assigning `state` variables directly with struct constructors wastes gas

Examples:
```solidity
    // stateStruct = TestStruct(20, 20); // 4633 gas
    // stateStruct = TestStruct({ var1: 20, var2: 20 }); // 4633 gas
    // stateStruct.var1 = 20;
    // stateStruct.var2 = 20; // 4562 gas
```

Test case:
```diff
File: src/dao/Proposals.sol
-		// _lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower );
+		_lastUserVoteForBallot[ballotID][msg.sender].vote = Vote(vote);
+		_lastUserVoteForBallot[ballotID][msg.sender].votingPower = userVotingPower;
```

Result: 
```bash
	testRevertStep6() (gas: -7815 (-0.017%)) 
	testRevertStep1() (gas: -7815 (-0.017%)) 
	testRevertStep10() (gas: -7815 (-0.017%)) 
	testRevertStep5() (gas: -7815 (-0.017%)) 
	testRevertStep9() (gas: -7815 (-0.017%)) 
	testRevertStep11() (gas: -7815 (-0.017%)) 
	testRevertStep7() (gas: -7815 (-0.017%)) 
	testRevertStep3() (gas: -7815 (-0.017%)) 
	testRevertStep4() (gas: -7815 (-0.017%)) 
	testRevertStep8() (gas: -7815 (-0.017%)) 
	testRevertStep2() (gas: -7815 (-0.017%)) 
	testParameterBallotVoting() (gas: 138 (0.018%)) 
	testTotalVotesCastForBallot() (gas: 138 (0.020%)) 
	testApprovalBallotVoting() (gas: 230 (0.023%)) 
	testWinningParameterVote() (gas: 184 (0.024%)) 
	Overall gas change: -79517 (-0.006%)
```

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: src/dao/Proposals.sol

109: ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime )
290: _lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower )
```
[109](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L109) | [290](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L290)

```solidity
File: src/pools/PoolsConfig.sol

54: underlyingPoolTokens[poolID] = TokenPair(tokenA, tokenB)
```
[54](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L54)

```solidity
File: src/pools/PoolStats.sol

96: _arbitrageIndicies[poolID] = ArbitrageIndicies(poolIndex1, poolIndex2, poolIndex3)
```
[96](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L96)
</details>


### [G-06] Using `delete` on a `uint/int` variable is cheaper than assigning it to `0`

Result:
```bash
	testPerformUpkeepMaxGas() (gas: -396 (-0.000%)) 
	testPerformUpkeepZeroPrice() (gas: -14 (-0.000%)) 
	testComprehensivePerformUpkeep() (gas: -15 (-0.000%)) 
	testPerformUpkeepMaxGas() (gas: -29 (-0.001%)) 
	testDoublePerformUpkeep() (gas: -29 (-0.001%)) 
	testRevertStep7() (gas: -400 (-0.001%)) 
	testRevertStep6() (gas: -418 (-0.001%)) 
	testRevertStep1() (gas: -418 (-0.001%)) 
	testRevertStep10() (gas: -418 (-0.001%)) 
	testRevertStep5() (gas: -418 (-0.001%)) 
	testRevertStep9() (gas: -418 (-0.001%)) 
	testRevertStep11() (gas: -418 (-0.001%)) 
	testRevertStep3() (gas: -418 (-0.001%)) 
	testRevertStep4() (gas: -418 (-0.001%)) 
	testRevertStep8() (gas: -418 (-0.001%)) 
	testRevertStep2() (gas: -418 (-0.001%)) 
	testComprehensive() (gas: -36 (-0.001%)) 
	testSuccessStep7() (gas: -15 (-0.001%)) 
	testProcessRewardsFromPOL() (gas: -18 (-0.002%)) 
	testPerformUpkeep() (gas: -18 (-0.002%)) 
	testUserCooldowns() (gas: -5 (-0.002%)) 
	testProfitsForPoolsSimple() (gas: -24 (-0.003%)) 
	testUpdateLastUpkeepTime() (gas: -90 (-0.006%)) 
	testLiveBalanceCheck() (gas: -400 (-0.012%)) 
	testAddRemoveLiquidityBeforeExchangeLive() (gas: 400 (0.013%)) 
	testSetContracts() (gas: 400 (0.013%)) 
	testPriceFeedGas() (gas: -36 (-0.041%)) 
	testUserCooldownsWhenNotInPools() (gas: -10 (-0.052%)) 
	testUniswap() (gas: -54 (-0.066%)) 
	testCoreUniswapFeedConstructor(address,address,address) (gas: -2231 (-0.180%)) 
	testCoreUniswapFeedInitialization() (gas: -2200 (-0.183%)) 
	testUnimodalHypothesis(uint256,uint256,uint256,uint256,uint256,uint256,uint256) (gas: -28 (-1.772%)) 
	Overall gas change: -9428 (-0.001%)
```

<details>
<summary><i>7 issue instances in 6 files:</i></summary>

```solidity
File: src/pools/PoolStats.sol

54: _arbitrageProfits[ poolIDs[i] ] = 0
```
[54](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L54)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

48: price = 0
```
[48](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L48)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

54: secondsAgo[1] = 0
```
[54](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L54)

```solidity
File: src/stable/CollateralAndLiquidity.sol

184: usdsBorrowedByUsers[wallet] = 0
```
[184](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L184)

```solidity
File: src/stable/Liquidizer.sol

113: usdsThatShouldBeBurned = 0
```
[113](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L113)

```solidity
File: src/staking/StakingRewards.sol

151: claimableRewards = 0
301: cooldowns[i] = 0
```
[151](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L151) | [301](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L301)
</details>


### [G-07] Assembly: Check `msg.sender` using xor and the scratch space

See [this](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender) prior finding for details on the conversion

<details>
<summary><i>32 issue instances in 17 files:</i></summary>

```solidity
File: src/dao/DAO.sol

297: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );
318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );
329: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );
362: require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```
[297](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297) | [318](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L318) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L329) | [362](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L362)

```solidity
File: src/dao/Proposals.sol

86: if ( msg.sender != address(exchangeConfig.dao() ) )
124: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );
132: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
```
[86](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L86) | [124](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L124) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L132)

```solidity
File: src/launch/Airdrop.sol

48: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
58: require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
```
[48](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48) | [58](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L58)

```solidity
File: src/launch/InitialDistribution.sol

52: require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52)

```solidity
File: src/pools/PoolStats.sol

49: require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```
[49](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L49)

```solidity
File: src/pools/Pools.sol

72: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );
142: require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
172: require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
```
[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L142) | [172](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L172)

```solidity
File: src/rewards/Emissions.sol

42: require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42)

```solidity
File: src/rewards/RewardsEmitter.sol

84: require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
```
[84](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L84)

```solidity
File: src/rewards/SaltRewards.sol

108: require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );
119: require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
```
[108](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L119)

```solidity
File: src/stable/USDS.sol

42: require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L42)

```solidity
File: src/stable/CollateralAndLiquidity.sol

142: require( wallet != msg.sender, "Cannot liquidate self" );
```
[142](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L142)

```solidity
File: src/stable/Liquidizer.sol

83: require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
134: require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83) | [134](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L134)

```solidity
File: src/staking/StakingRewards.sol

65: if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
105: if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
```
[65](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L65) | [105](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L105)

```solidity
File: src/staking/Staking.sol

90: require( msg.sender == u.wallet, "Sender is not the original staker" );
106: require( msg.sender == u.wallet, "Sender is not the original staker" );
132: require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );
```
[90](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L90) | [106](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L106) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L132)

```solidity
File: src/ManagedWallet.sol

44: require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
61: require( msg.sender == confirmationWallet, "Invalid sender" );
76: require( msg.sender == proposedMainWallet, "Invalid sender" );
```
[44](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44) | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L61) | [76](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L76)

```solidity
File: src/AccessManager.sol

43: require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43)

```solidity
File: src/Upkeep.sol

97: require(msg.sender == address(this), "Only callable from within the same contract");
```
[97](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97)
</details>


### [G-08] Use `revert()` to gain maximum gas savings

If you dont need Error messages, or you want gain maximum gas savings - `revert()` is a cheapest way to revert transaction in terms of gas.
```solidity
    revert(); // 117 gas 
    require(false); // 132 gas
    revert CustomError(); // 157 gas
    assert(false); // 164 gas
    revert("Custom Error"); // 406 gas
    require(false, "Custom Error"); // 421 gas
```


<details>
<summary><i>139 issue instances in 23 files:</i></summary>

```solidity
File: src/dao/DAO.sol

247: require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" )
251: require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" )
281: require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" )
297: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" )
318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" )
329: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" )
362: require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" )
```
[247](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247) | [251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L251) | [281](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L281) | [297](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297) | [318](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L318) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L329) | [362](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L362)

```solidity
File: src/dao/Proposals.sol

83: require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" )
92: require( requiredXSalt > 0, "requiredXSalt cannot be zero" )
95: require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" )
98: require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" )
102: require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" )
103: require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" )
124: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" )
132: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" )
164: require( address(token) != address(0), "token cannot be address(0)" )
165: require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" )
167: require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" )
168: require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" )
169: require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" )
182: require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" )
183: require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" )
184: require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" )
185: require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" )
186: require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" )
187: require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" )
198: require( wallet != address(0), "Cannot send SALT to address(0)" )
203: require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" )
215: require( contractAddress != address(0), "Contract address cannot be address(0)" )
224: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" )
233: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" )
242: require( newAddress != address(0), "Proposed address cannot be address(0)" )
251: require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" )
264: require( ballot.ballotIsLive, "The specified ballot is not open for voting" )
268: require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" )
270: require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" )
277: require( userVotingPower > 0, "Staked SALT required to vote" )
321: require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" )
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L83) | [92](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L92) | [95](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L95) | [98](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L98) | [102](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L102) | [103](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L103) | [124](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L124) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L132) | [164](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L164) | [165](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L165) | [167](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L167) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L168) | [169](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L169) | [182](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L182) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L183) | [184](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L184) | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L185) | [186](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L186) | [187](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L187) | [198](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L198) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L203) | [215](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L215) | [224](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L224) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L233) | [242](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L242) | [251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251) | [264](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L264) | [268](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L268) | [270](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L270) | [277](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L277) | [321](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L321)

```solidity
File: src/launch/Airdrop.sol

48: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" )
49: require( ! claimingAllowed, "Cannot authorize after claiming is allowed" )
58: require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" )
59: require( ! claimingAllowed, "Claiming is already allowed" )
60: require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.")
76: require( claimingAllowed, "Claiming is not allowed yet" )
77: require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" )
78: require( ! claimed[msg.sender], "Wallet already claimed the airdrop" )
```
[48](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48) | [49](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L49) | [58](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L58) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L59) | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L60) | [76](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L76) | [77](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L77) | [78](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L78)

```solidity
File: src/launch/BootstrapBallot.sol

36: require( ballotDuration > 0, "ballotDuration cannot be zero" )
50: require( ! hasVoted[msg.sender], "User already voted" )
54: require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" )
71: require( ! ballotFinalized, "Ballot has already been finalized" )
72: require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" )
```
[36](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L36) | [50](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L50) | [54](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L54) | [71](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L71) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L72)

```solidity
File: src/launch/InitialDistribution.sol

52: require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" )
53: require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" )
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52) | [53](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L53)

```solidity
File: src/pools/PoolsConfig.sol

47: require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" )
48: require(tokenA != tokenB, "tokenA and tokenB cannot be the same token")
136: require(address(pair.tokenA) != address(0) && address(pair.tokenB) != address(0), "This poolID does not exist")
```
[47](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L47) | [48](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L48) | [136](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L136)

```solidity
File: src/pools/PoolStats.sol

49: require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" )
```
[49](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L49)

```solidity
File: src/pools/Pools.sol

72: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" )
83: require(block.timestamp <= deadline, "TX EXPIRED")
142: require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" )
143: require( exchangeIsLive, "The exchange is not yet live" )
144: require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" )
146: require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" )
147: require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" )
158: require( addedLiquidity >= minLiquidityReceived, "Too little liquidity received" )
172: require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" )
173: require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" )
187: require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal")
193: require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" )
207: require( amount > PoolUtils.DUST, "Deposit amount too small")
221: require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" )
222: require( amount > PoolUtils.DUST, "Withdraw amount too small")
243: require((reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves before swap")
271: require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap")
353: require( swapAmountOut >= minAmountOut, "Insufficient resulting token amount" )
371: require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" )
```
[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72) | [83](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L83) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L142) | [143](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L143) | [144](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L144) | [146](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L146) | [147](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L147) | [158](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L158) | [172](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L172) | [173](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L173) | [187](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L187) | [193](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L193) | [207](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L207) | [221](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L221) | [222](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222) | [243](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L243) | [271](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L271) | [353](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L353) | [371](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L371)

```solidity
File: src/price_feed/PriceAggregator.sol

39: require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" )
183: require (price != 0, "Invalid BTC price" )
195: require (price != 0, "Invalid ETH price" )
```
[39](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L39) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L183) | [195](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L195)

```solidity
File: src/rewards/Emissions.sol

42: require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" )
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42)

```solidity
File: src/rewards/RewardsEmitter.sol

63: require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" )
84: require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" )
```
[63](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L63) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L84)

```solidity
File: src/rewards/SaltRewards.sol

60: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" )
108: require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" )
119: require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" )
120: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" )
```
[60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L60) | [108](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L119) | [120](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L120)

```solidity
File: src/stable/USDS.sol

42: require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" )
43: require( amount > 0, "Cannot mint zero USDS" )
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L42) | [43](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L43)

```solidity
File: src/stable/CollateralAndLiquidity.sol

83: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" )
84: require( collateralToWithdraw <= maxWithdrawableCollateral(msg.sender), "Excessive collateralToWithdraw" )
97: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" )
98: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" )
99: require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" )
117: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" )
118: require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" )
119: require( amountRepaid > 0, "Cannot repay zero amount" )
142: require( wallet != msg.sender, "Cannot liquidate self" )
145: require( canUserBeLiquidated(wallet), "User cannot be liquidated" )
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L84) | [97](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L97) | [98](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L98) | [99](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L99) | [117](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L117) | [118](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L118) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L119) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L142) | [145](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L145)

```solidity
File: src/stable/Liquidizer.sol

83: require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" )
134: require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" )
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83) | [134](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L134)

```solidity
File: src/staking/Liquidity.sol

42: require(block.timestamp <= deadline, "TX EXPIRED")
85: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" )
124: require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" )
148: require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be deposited via Liquidity.depositLiquidityAndIncreaseShare" )
159: require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be withdrawn via Liquidity.withdrawLiquidityAndClaim" )
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L42) | [85](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L85) | [124](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L124) | [148](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L148) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L159)

```solidity
File: src/staking/StakingRewards.sol

59: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" )
60: require( increaseShareAmount != 0, "Cannot increase zero share" )
67: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" )
99: require( decreaseShareAmount != 0, "Cannot decrease zero share" )
102: require( decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share" )
107: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" )
190: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" )
```
[59](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L59) | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L60) | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L67) | [99](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L99) | [102](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L102) | [107](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L107) | [190](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L190)

```solidity
File: src/staking/Staking.sol

43: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" )
62: require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" )
88: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" )
89: require( block.timestamp < u.completionTime, "Unstakes that have already completed cannot be cancelled" )
90: require( msg.sender == u.wallet, "Sender is not the original staker" )
104: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" )
105: require( block.timestamp >= u.completionTime, "Unstake has not completed yet" )
106: require( msg.sender == u.wallet, "Sender is not the original staker" )
132: require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" )
153: require(end >= start, "Invalid range: end cannot be less than start")
157: require(userUnstakes.length > end, "Invalid range: end is out of bounds")
158: require(start < userUnstakes.length, "Invalid range: start is out of bounds")
204: require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" )
205: require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" )
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L43) | [62](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L62) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L88) | [89](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L89) | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L90) | [104](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L104) | [105](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L105) | [106](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L106) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L132) | [153](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L153) | [157](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L157) | [158](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L158) | [204](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L204) | [205](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L205)

```solidity
File: src/ManagedWallet.sol

44: require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" )
45: require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" )
46: require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" )
49: require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." )
61: require( msg.sender == confirmationWallet, "Invalid sender" )
76: require( msg.sender == proposedMainWallet, "Invalid sender" )
77: require( block.timestamp >= activeTimelock, "Timelock not yet completed" )
```
[44](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44) | [45](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L45) | [46](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L46) | [49](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L49) | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L61) | [76](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L76) | [77](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L77)

```solidity
File: src/AccessManager.sol

43: require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" )
63: require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" )
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43) | [63](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L63)

```solidity
File: src/SigningTools.sol

13: require( signature.length == 65, "Invalid signature length" )
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L13)

```solidity
File: src/Upkeep.sol

97: require(msg.sender == address(this), "Only callable from within the same contract")
```
[97](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97)

```solidity
File: src/ExchangeConfig.sol

51: require( address(dao) == address(0), "setContracts can only be called once" )
64: require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" )
```
[51](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51) | [64](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L64)
</details>


### [G-09] Using `constants` instead of `enum` can save gas

`Enum` is expensive and it is [more efficient to use constants](https://www.codehawks.com/finding/clm84992q02j9w9ruebun36d9) instead.
An illustrative example of this approach can be found in the [ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/181d518609a9f006fcb97af63e6952e603cf100e/contracts/utils/ReentrancyGuard.sol#L34-L35)

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/dao/Parameters.sol

14: enum ParameterTypes {

		// PoolsConfig
		maximumWhitelistedPools,
		maximumInternalSwapPercentTimes1000,

		// StakingConfig
		minUnstakeWeeks,
		maxUnstakeWeeks,
		minUnstakePercent,
		modificationCooldown,

		// RewardsConfig
    	rewardsEmitterDailyPercentTimes1000,
		emissionsWeeklyPercentTimes1000,
		stakingRewardsPercent,
		percentRewardsSaltUSDS,

		// StableConfig
		rewardPercentForCallingLiquidation,
		maxRewardValueForCallingLiquidation,
		minimumCollateralValueForBorrowing,
		initialCollateralRatioPercent,
		minimumCollateralRatioPercent,
		percentArbitrageProfitsForStablePOL,

		// DAOConfig
		bootstrappingRewards,
		percentPolRewardsBurned,
		baseBallotQuorumPercentTimes1000,
		ballotDuration,
		requiredProposalPercentStakeTimes1000,
		maxPendingTokensForWhitelisting,
		arbitrageProfitsPercentPOL,
		upkeepRewardPercent,

		// PriceAggregator
		maximumPriceFeedPercentDifferenceTimes1000,
		setPriceFeedCooldown
		}
```
[14](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L14)
</details>


### [G-10] Optimize Increment and Decrement in loops with `unchecked` keyword

Use `unchecked{i++}` or `unchecked{++i}` instead of `i++` or `++i` when it is not possible for them to overflow.
This is applicable for Solidity version `0.8.0` or higher to `0.8.23` and saves 30-40 gas per loop.

<details>
<summary><i>21 issue instances in 8 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

115: for( uint256 i = 0; i < 8; i++ )
				{
```
[115](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L115)

```solidity
File: src/dao/Proposals.sol

423: for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
```
[423](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423)

```solidity
File: src/pools/PoolStats.sol

53: for( uint256 i = 0; i < poolIDs.length; i++ )
			_arbitrageProfits[ poolIDs[i] ] = 0;
		}


	// The index of pool tokenA/tokenB within the whitelistedPools array.
	// Should always find a value as only whitelisted pools are used in the arbitrage path.
	// Returns uint64.max in the event of failed lookup
	function _poolIndex( IERC20 tokenA, IERC20 tokenB, bytes32[] memory poolIDs ) internal pure returns (uint64 index)
		{
65: for( uint256 i = 0; i < poolIDs.length; i++ )
			{
81: for( uint256 i = 0; i < poolIDs.length; i++ )
			{
106: for( uint256 i = 0; i < poolIDs.length; i++ )
			{
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L53) | [65](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L65) | [81](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L81) | [106](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L106)

```solidity
File: src/rewards/RewardsEmitter.sol

60: for( uint256 i = 0; i < addedRewards.length; i++ )
			{
116: for( uint256 i = 0; i < poolIDs.length; i++ )
			{
```
[60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L60) | [116](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L116)

```solidity
File: src/rewards/SaltRewards.sol

64: for( uint256 i = 0; i < addedRewards.length; i++ )
			{
87: for( uint256 i = 0; i < addedRewards.length; i++ )
			addedRewards[i] = AddedReward( poolIDs[i], amountPerPool );

		// Send the liquidity bootstrap rewards to the liquidityRewardsEmitter
		liquidityRewardsEmitter.addSALTRewards( addedRewards );
		}


	function _sendInitialStakingRewards( uint256 stakingBootstrapAmount ) internal
		{
```
[64](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L64) | [87](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L87)

```solidity
File: src/stable/CollateralAndLiquidity.sol

321: for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
321: for ( uint256 i = startIndex; i <= endIndex; i++ )
				{
338: for ( uint256 i = 0; i < count; i++ )
			resizedLiquidatableUsers[i] = liquidatableUsers[i];

		return resizedLiquidatableUsers;
		}


	function findLiquidatableUsers() external view returns (address[] memory)
		{
```
[321](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321) | [321](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L321) | [338](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L338)

```solidity
File: src/staking/StakingRewards.sol

152: for( uint256 i = 0; i < poolIDs.length; i++ )
			{
185: for( uint256 i = 0; i < addedRewards.length; i++ )
			{
216: for( uint256 i = 0; i < shares.length; i++ )
			shares[i] = totalShares[ poolIDs[i] ];
		}


	// Returns the total rewards for specified pools.
	function totalRewardsForPools( bytes32[] calldata poolIDs ) external view returns (uint256[] memory rewards)
		{
226: for( uint256 i = 0; i < rewards.length; i++ )
			rewards[i] = totalRewards[ poolIDs[i] ];
		}


	// Returns the user's pending rewards for a specified pool.
	function userRewardForPool( address wallet, bytes32 poolID ) public view returns (uint256)
		{
260: for( uint256 i = 0; i < rewards.length; i++ )
			rewards[i] = userRewardForPool( wallet, poolIDs[i] );
		}


	// Get the user's shares for a specified pool.
	function userShareForPool( address wallet, bytes32 poolID ) public view returns (uint256)
		{
277: for( uint256 i = 0; i < shares.length; i++ )
			shares[i] = _userShareInfo[wallet][ poolIDs[i] ].userShare;
		}


	// Get the user's virtual rewards for a specified pool.
	function userVirtualRewardsForPool( address wallet, bytes32 poolID ) public view returns (uint256)
		{
296: for( uint256 i = 0; i < cooldowns.length; i++ )
			{
```
[152](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L152) | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L185) | [216](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L216) | [226](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L226) | [260](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L260) | [277](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L277) | [296](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L296)

```solidity
File: src/staking/Staking.sol

163: for(uint256 i = start; i <= end; i++)
            unstakes[index++] = _unstakesByID[ userUnstakes[i]];

        return unstakes;
    }


	// Retrieve all pending unstakes associated with a user.
	function unstakesForUser( address user ) external view returns (Unstake[] memory)
		{
```
[163](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L163)
</details>


### [G-11] Reduce deployment costs by tweaking contracts' metadata

The Solidity compiler appends 53 bytes of metadata to the smart contract code which translates to an extra 10,600 gas (200 per bytecode) + the calldata cost (16 gas per non-zero bytes, 4 gas per zero-byte).
This translates to up to 848 additional gas in calldata cost.
One way to reduce this cost is by optimizing the IPFS hash that gets appended to the smart contract code.

Why is this important?
- The metadata adds an extra 53 bytes, resulting in an additional 10,600 gas cost for deployment.
- It also incurs up to 848 additional gas in calldata cost.

Options to Reduce Gas:
1. Use the `--no-cbor-metadata` compiler option to exclude metadata, but this might affect contract verification.
2. Mine for code comments that lead to an IPFS hash with more zeros, reducing calldata costs.

<details>
<summary><i>35 issue instances in 35 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L1)

```solidity
File: src/dao/DAO.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L1)

```solidity
File: src/dao/DAOConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L1)

```solidity
File: src/dao/Proposals.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L1)

```solidity
File: src/dao/Parameters.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol#L1)

```solidity
File: src/launch/Airdrop.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L1)

```solidity
File: src/launch/BootstrapBallot.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L1)

```solidity
File: src/launch/InitialDistribution.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L1)

```solidity
File: src/pools/PoolUtils.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L1)

```solidity
File: src/pools/PoolsConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L1)

```solidity
File: src/pools/PoolStats.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L1)

```solidity
File: src/pools/Pools.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L1)

```solidity
File: src/pools/PoolMath.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L1)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L1)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L1)

```solidity
File: src/price_feed/PriceAggregator.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L1)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L1)

```solidity
File: src/rewards/RewardsConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol#L1)

```solidity
File: src/rewards/Emissions.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L1)

```solidity
File: src/rewards/RewardsEmitter.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L1)

```solidity
File: src/rewards/SaltRewards.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L1)

```solidity
File: src/stable/USDS.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L1)

```solidity
File: src/stable/StableConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L1)

```solidity
File: src/stable/CollateralAndLiquidity.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L1)

```solidity
File: src/stable/Liquidizer.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L1)

```solidity
File: src/staking/StakingConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol#L1)

```solidity
File: src/staking/Liquidity.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L1)

```solidity
File: src/staking/StakingRewards.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L1)

```solidity
File: src/staking/Staking.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L1)

```solidity
File: src/ManagedWallet.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L1)

```solidity
File: src/AccessManager.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L1)

```solidity
File: src/Salt.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L1)

```solidity
File: src/SigningTools.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L1)

```solidity
File: src/Upkeep.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L1)

```solidity
File: src/ExchangeConfig.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L1)
</details>


### [G-12] Consider using solady's `FixedPointMathLib`

Utilizing gas-optimized math functions from libraries like [Solady](https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol) can lead to more efficient smart contracts.
This is particularly beneficial in contracts where these operations are frequently used.

<details>
<summary><i>200 issue instances in 21 files:</i></summary>

```solidity
File: src/arbitrage/ArbitrageSearch.sol

68: uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
69: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
70: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
82: amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
83: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
84: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
129: uint256 amountOut = (reservesA1 * bestArbAmountIn) / (reservesA0 + bestArbAmountIn);
130: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
131: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
68: uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
69: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
70: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
82: amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
83: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
84: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
129: uint256 amountOut = (reservesA1 * bestArbAmountIn) / (reservesA0 + bestArbAmountIn);
130: amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
131: amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);
```
[68](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L68) | [69](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L69) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L70) | [82](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L82) | [83](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L84) | [129](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L129) | [130](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L130) | [131](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L131) | [68](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L68) | [69](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L69) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L70) | [82](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L82) | [83](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L84) | [129](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L129) | [130](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L130) | [131](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L131)

```solidity
File: src/dao/DAO.sol

247: require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
265: exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
348: uint256 saltToBurn = ( remainingSALT * daoConfig.percentPolRewardsBurned() ) / 100;
369: uint256 liquidityToWithdraw = (liquidityHeld * percentToLiquidate) / 100;
341: uint256 amountToSendToTeam = claimedSALT / 10;
348: uint256 saltToBurn = ( remainingSALT * daoConfig.percentPolRewardsBurned() ) / 100;
369: uint256 liquidityToWithdraw = (liquidityHeld * percentToLiquidate) / 100;
```
[247](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247) | [265](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L265) | [348](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L348) | [369](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L369) | [341](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L341) | [348](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L348) | [369](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L369)

```solidity
File: src/dao/DAOConfig.sol

40: uint256 public baseBallotQuorumPercentTimes1000 = 10 * 1000; // Default 10% of the total amount of SALT staked with a 1000x multiplier
102: if (baseBallotQuorumPercentTimes1000 < 20 * 1000)
107: if (baseBallotQuorumPercentTimes1000 > 5 * 1000 )
```
[40](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L40) | [102](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L102) | [107](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol#L107)

```solidity
File: src/dao/Proposals.sol

90: uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
90: uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
202: uint256 maxSendable = balance * 5 / 100;
324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
335: uint256 minimumQuorum = totalSupply * 5 / 1000;
90: uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
202: uint256 maxSendable = balance * 5 / 100;
324: requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
326: requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
329: requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
335: uint256 minimumQuorum = totalSupply * 5 / 1000;
```
[90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L90) | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L90) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L202) | [324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324) | [324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324) | [324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329) | [335](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L335) | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L90) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L202) | [324](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L324) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L326) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L329) | [335](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L335)

```solidity
File: src/launch/Airdrop.sol

64: saltAmountForEachUser = saltBalance / numberAuthorized();
```
[64](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L64)

```solidity
File: src/launch/InitialDistribution.sol

53: require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
56: salt.safeTransfer( address(emissions), 52 * MILLION_ETHER );
59: salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );
62: salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );
65: salt.safeTransfer( address(airdrop), 5 * MILLION_ETHER );
72: salt.safeTransfer( address(saltRewards), 8 * MILLION_ETHER );
73: saltRewards.sendInitialSaltRewards(5 * MILLION_ETHER, poolIDs );
```
[53](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L53) | [56](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L56) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L59) | [62](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L62) | [65](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L65) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L72) | [73](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L73)

```solidity
File: src/pools/PoolUtils.sol

61: uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
61: uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
61: uint256 maxAmountIn = reservesIn * maximumInternalSwapPercentTimes1000 / (100 * 1000);
```
[61](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L61) | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L61) | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolUtils.sol#L61)

```solidity
File: src/pools/Pools.sol

109: uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;
115: addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
132: addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
134: addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
179: reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
180: reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;
260: amountOut = reserve0 * amountIn / reserve1;
266: amountOut = reserve1 * amountIn / reserve0;
313: swapAmountInValueInETH = ( swapAmountIn * reservesWETH ) / reservesTokenIn;
109: uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;
115: addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
132: addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
134: addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
179: reclaimedA = ( reserves.reserve0 * liquidityToRemove ) / totalLiquidity;
180: reclaimedB = ( reserves.reserve1 * liquidityToRemove ) / totalLiquidity;
260: amountOut = reserve0 * amountIn / reserve1;
266: amountOut = reserve1 * amountIn / reserve0;
313: swapAmountInValueInETH = ( swapAmountIn * reservesWETH ) / reservesTokenIn;
```
[109](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L109) | [115](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L115) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L132) | [134](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L134) | [179](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L179) | [180](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L180) | [260](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L260) | [266](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L266) | [313](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L313) | [109](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L109) | [115](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L115) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L132) | [134](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L134) | [179](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L179) | [180](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L180) | [260](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L260) | [266](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L266) | [313](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L313)

```solidity
File: src/pools/PoolMath.sol

15: k = r0 * r1
21: s1 = r1 - r0 * r1 / (r0 + s0)
184: uint256 B = 2 * r0;
192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
193: uint256 discriminant = B * B + 4 * A * C;
193: uint256 discriminant = B * B + 4 * A * C;
203: swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );
215: if ( zapAmountA * reserveB > reserveA * zapAmountB )
215: if ( zapAmountA * reserveB > reserveA * zapAmountB )
219: if ( zapAmountA * reserveB < reserveA * zapAmountB )
219: if ( zapAmountA * reserveB < reserveA * zapAmountB )
18: s1 = r1 - k / (r0 + s0)
21: s1 = r1 - r0 * r1 / (r0 + s0)
24: (r0 + s0) / ( r1 - s1)
31: a0 / a1 = (r0 + s0) / ( r1 - s1)
31: a0 / a1 = (r0 + s0) / ( r1 - s1)
34: (z0 - s0) / ( z1 + s1) = (r0 + s0) / ( r1 - s1)
34: (z0 - s0) / ( z1 + s1) = (r0 + s0) / ( r1 - s1)
41: (c-x)/(d+y) = (a+x)/(b-y)
41: (c-x)/(d+y) = (a+x)/(b-y)
44: y = b - ab/(a+x)
47: (c-x)/(d+y) = (a+x)/(b-y)
47: (c-x)/(d+y) = (a+x)/(b-y)
68: x(b+d) + b(a+c) - ab(a+c)/(a+x) = bc - ad
95: xx + x(2a) + (aad-abc)/(b+d) = 0
101: C = a(ad - bc)/(b+d)
109: C = r0(r0z1 - r1z0)/(r1 + z1)
192: uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
203: swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );
```
[15](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L15) | [21](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L21) | [184](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L184) | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192) | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192) | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192) | [193](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L193) | [193](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L193) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L203) | [215](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L215) | [215](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L215) | [219](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L219) | [219](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L219) | [18](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L18) | [21](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L21) | [24](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L24) | [31](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L31) | [31](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L31) | [34](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L34) | [34](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L34) | [41](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L41) | [41](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L41) | [44](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L44) | [47](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L47) | [47](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L47) | [68](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L68) | [95](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L95) | [101](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L101) | [109](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L109) | [192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L192) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L203)

```solidity
File: src/price_feed/CoreChainlinkFeed.sol

59: return uint256(price) * 10**10;
59: return uint256(price) * 10**10;
```
[59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L59) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L59)

```solidity
File: src/price_feed/CoreUniswapFeed.sol

70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
72: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
112: return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;
59: int24 tick = int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)));
70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
72: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
107: uniswapWBTC_WETH = 10**36 / uniswapWBTC_WETH;
110: uniswapWETH_USDC = 10**36 / uniswapWETH_USDC;
112: return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;
126: return 10**36 / uniswapWETH_USDC;
67: return FullMath.mulDiv( 10 ** ( 18 + decimals1 - decimals0 ), FixedPoint96.Q96, p );
70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
70: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );
72: return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
107: uniswapWBTC_WETH = 10**36 / uniswapWBTC_WETH;
110: uniswapWETH_USDC = 10**36 / uniswapWETH_USDC;
112: return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;
126: return 10**36 / uniswapWETH_USDC;
```
[70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L72) | [112](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L59) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L72) | [107](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L107) | [110](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L110) | [112](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L126) | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L67) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70) | [70](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L70) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L72) | [107](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L107) | [110](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L110) | [112](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112) | [126](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L126)

```solidity
File: src/price_feed/PriceAggregator.sol

142: if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
139: uint256 averagePrice = ( priceA + priceB ) / 2;
142: if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
```
[142](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L142) | [139](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L139) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L142)

```solidity
File: src/price_feed/CoreSaltyFeed.sol

40: return ( reservesUSDS * 10**8 ) / reservesWBTC;
52: return ( reservesUSDS * 10**18 ) / reservesWETH;
40: return ( reservesUSDS * 10**8 ) / reservesWBTC;
52: return ( reservesUSDS * 10**18 ) / reservesWETH;
52: return ( reservesUSDS * 10**18 ) / reservesWETH;
```
[40](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L40) | [52](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L52) | [40](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L40) | [52](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L52) | [52](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L52)

```solidity
File: src/rewards/Emissions.sol

55: uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
55: uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
55: uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
55: uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );
```
[55](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55) | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55) | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55) | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L55)

```solidity
File: src/rewards/RewardsEmitter.sol

112: uint256 numeratorMult = timeSinceLastUpkeep * rewardsConfig.rewardsEmitterDailyPercentTimes1000();
113: uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000
113: uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000
113: uint256 denominatorMult = 1 days * 100000; // simplification of numberSecondsInOneDay * (100 percent) * 1000
121: uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;
```
[112](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L112) | [113](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L113) | [113](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L113) | [113](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L113) | [121](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L121)

```solidity
File: src/rewards/SaltRewards.sol

67: uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;
139: uint256 directRewardsForSaltUSDS = ( saltRewardsToDistribute * rewardsConfig.percentRewardsSaltUSDS() ) / 100;
143: uint256 stakingRewardsAmount = ( remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100;
67: uint256 rewardsForPool = ( liquidityRewardsAmount * profitsForPools[i] ) / totalProfits;
84: uint256 amountPerPool = liquidityBootstrapAmount / poolIDs.length; // poolIDs.length is guaranteed to not be zero
139: uint256 directRewardsForSaltUSDS = ( saltRewardsToDistribute * rewardsConfig.percentRewardsSaltUSDS() ) / 100;
143: uint256 stakingRewardsAmount = ( remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100;
```
[67](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L67) | [139](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L139) | [143](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L143) | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L67) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L84) | [139](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L139) | [143](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L143)

```solidity
File: src/stable/CollateralAndLiquidity.sol

159: uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
160: uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;
167: rewardedWBTC = (rewardedWBTC * maxRewardValue) / rewardValue;
168: rewardedWETH = (rewardedWETH * maxRewardValue) / rewardValue;
202: uint256 btcValue = ( amountBTC * btcPrice ) / wbtcTenToTheDecimals;
203: uint256 ethValue = ( amountETH * ethPrice ) / wethTenToTheDecimals;
232: uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares;
233: uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares;
261: return userCollateralAmount * maxWithdrawableValue / userCollateralValue;
280: uint256 maxBorrowableAmount = ( userCollateralValue * 100 ) / stableConfig.initialCollateralRatioPercent();
307: return (( userCollateralValue * 100 ) / usdsBorrowedAmount) < stableConfig.minimumCollateralRatioPercent();
329: uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
159: uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
160: uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;
167: rewardedWBTC = (rewardedWBTC * maxRewardValue) / rewardValue;
168: rewardedWETH = (rewardedWETH * maxRewardValue) / rewardValue;
202: uint256 btcValue = ( amountBTC * btcPrice ) / wbtcTenToTheDecimals;
203: uint256 ethValue = ( amountETH * ethPrice ) / wethTenToTheDecimals;
232: uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares;
233: uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares;
250: uint256 requiredCollateralValueAfterWithdrawal = ( usdsBorrowedByUsers[wallet] * stableConfig.initialCollateralRatioPercent() ) / 100;
261: return userCollateralAmount * maxWithdrawableValue / userCollateralValue;
280: uint256 maxBorrowableAmount = ( userCollateralValue * 100 ) / stableConfig.initialCollateralRatioPercent();
307: return (( userCollateralValue * 100 ) / usdsBorrowedAmount) < stableConfig.minimumCollateralRatioPercent();
326: uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
329: uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
63: wbtcTenToTheDecimals = 10 ** IERC20Metadata(address(wbtc)).decimals();
64: wethTenToTheDecimals = 10 ** IERC20Metadata(address(weth)).decimals();
```
[159](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L159) | [160](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L160) | [167](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L167) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L168) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L202) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L203) | [232](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L233) | [261](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L261) | [280](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L280) | [307](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L307) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L329) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L159) | [160](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L160) | [167](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L167) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L168) | [202](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L202) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L203) | [232](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L233) | [250](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L250) | [261](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L261) | [280](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L280) | [307](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L307) | [326](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L329) | [63](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L63) | [64](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L64)

```solidity
File: src/staking/StakingRewards.sol

118: uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;
78: if ( existingTotalShares != 0 ) // prevent / 0
114: uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];
118: uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;
243: uint256 rewardsShare = ( totalRewards[poolID] * user.userShare ) / totalShares[poolID];
```
[118](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L118) | [78](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L78) | [114](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L114) | [118](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L118) | [243](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L243)

```solidity
File: src/staking/Staking.sol

65: uint256 completionTime = block.timestamp + numWeeks * ( 1 weeks );
210: uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
210: uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
210: uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
211: return numerator / ( 100 * unstakeRange );
211: return numerator / ( 100 * unstakeRange );
```
[65](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L65) | [210](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L210) | [210](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L210) | [210](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L210) | [211](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L211) | [211](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L211)

```solidity
File: src/Salt.sol

13: uint256 public constant INITIAL_SUPPLY = 100 * MILLION_ETHER ;
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol#L13)

```solidity
File: src/Upkeep.sol

119: uint256 rewardAmount = withdrawnAmount * daoConfig.upkeepRewardPercent() / 100;
152: uint256 amountOfWETH = wethBalance * stableConfig.percentArbitrageProfitsForStablePOL() / 100;
165: uint256 amountOfWETH = wethBalance * daoConfig.arbitrageProfitsPercentPOL() / 100;
289: return daoWETH * daoConfig.upkeepRewardPercent() / 100;
119: uint256 rewardAmount = withdrawnAmount * daoConfig.upkeepRewardPercent() / 100;
152: uint256 amountOfWETH = wethBalance * stableConfig.percentArbitrageProfitsForStablePOL() / 100;
165: uint256 amountOfWETH = wethBalance * daoConfig.arbitrageProfitsPercentPOL() / 100;
289: return daoWETH * daoConfig.upkeepRewardPercent() / 100;
```
[119](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L119) | [152](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L152) | [165](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L165) | [289](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L289) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L119) | [152](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L152) | [165](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L165) | [289](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L289)
</details>


### [G-13] Consider using OZ EnumerateSet in place of nested mappings

Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).

A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.

OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

<details>
<summary><i>5 issue instances in 4 files:</i></summary>

```solidity
File: src/dao/Proposals.sol

51: mapping(uint256=>mapping(Vote=>uint256)) private _votesCastForBallot;
55: mapping(uint256=>mapping(address=>UserVote)) private _lastUserVoteForBallot;
```
[51](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L51) | [55](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L55)

```solidity
File: src/pools/Pools.sol

49: mapping(address=>mapping(IERC20=>uint256)) private _userDeposits;
```
[49](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L49)

```solidity
File: src/staking/StakingRewards.sol

36: mapping(address=>mapping(bytes32=>UserShareInfo)) private _userShareInfo;
```
[36](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L36)

```solidity
File: src/AccessManager.sol

26: mapping(uint256 => mapping(address => bool)) private _walletsWithAccess;
```
[26](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L26)
</details>


### [G-14] Optimize `require/revert` Statements with Assembly

Using inline assembly for revert statements in Solidity can offer gas optimizations.
The typical `require` or `revert` statements in Solidity perform additional memory operations and type checks which can be avoided by using low-level assembly code.

In certain contracts, particularly those that might revert often or are otherwise sensitive to gas costs, using assembly to handle reversion can offer meaningful savings.
These savings primarily come from avoiding memory expansion costs and extra type checks that the Solidity compiler performs.

Example:
```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
function restrictedAction(uint256 num)  external {
    require(owner == msg.sender, "caller is not owner");
    specialNumber = num;
}

// calling restrictedAction(2) with a non-owner address: 23734
function restrictedAction(uint256 num)  external {
    assembly {
        if sub(caller(), sload(owner.slot)) {
            mstore(0x00, 0x20) // store offset to where length of revert message is stored
            mstore(0x20, 0x13) // store length (19)
            mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
            revert(0x00, 0x60) // revert with data
        }
    }
    specialNumber = num;
}
```

<details>
<summary><i>139 issue instances in 23 files:</i></summary>

```solidity
File: src/dao/DAO.sol

247: require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );
251: require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );
281: require( proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized" );
297: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.withdrawArbitrageProfits is only callable from the Upkeep contract" );
318: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.formPOL is only callable from the Upkeep contract" );
329: require( msg.sender == address(exchangeConfig.upkeep()), "DAO.processRewardsFromPOL is only callable from the Upkeep contract" );
362: require(msg.sender == address(liquidizer), "DAO.withdrawProtocolOwnedLiquidity is only callable from the Liquidizer contract" );
```
[247](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247) | [251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L251) | [281](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L281) | [297](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L297) | [318](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L318) | [329](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L329) | [362](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L362)

```solidity
File: src/dao/Proposals.sol

83: require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );
92: require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
95: require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );
98: require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
102: require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
103: require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );
124: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can create a confirmation proposal" );
132: require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );
164: require( address(token) != address(0), "token cannot be address(0)" );
165: require( token.totalSupply() < type(uint112).max, "Token supply cannot exceed uint112.max" ); // 5 quadrillion max supply with 18 decimals of precision
167: require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );
168: require( poolsConfig.numberOfWhitelistedPools() < poolsConfig.maximumWhitelistedPools(), "Maximum number of whitelisted pools already reached" );
169: require( ! poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "The token has already been whitelisted" );
182: require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
183: require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
184: require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
185: require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
186: require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
187: require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
198: require( wallet != address(0), "Cannot send SALT to address(0)" );
203: require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );
215: require( contractAddress != address(0), "Contract address cannot be address(0)" );
224: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
233: require( bytes(country).length == 2, "Country must be an ISO 3166 Alpha-2 Code" );
242: require( newAddress != address(0), "Proposed address cannot be address(0)" );
251: require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
264: require( ballot.ballotIsLive, "The specified ballot is not open for voting" );
268: require( (vote == Vote.INCREASE) || (vote == Vote.DECREASE) || (vote == Vote.NO_CHANGE), "Invalid VoteType for Parameter Ballot" );
270: require( (vote == Vote.YES) || (vote == Vote.NO), "Invalid VoteType for Approval Ballot" );
277: require( userVotingPower > 0, "Staked SALT required to vote" );
321: require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L83) | [92](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L92) | [95](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L95) | [98](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L98) | [102](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L102) | [103](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L103) | [124](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L124) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L132) | [164](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L164) | [165](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L165) | [167](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L167) | [168](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L168) | [169](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L169) | [182](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L182) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L183) | [184](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L184) | [185](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L185) | [186](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L186) | [187](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L187) | [198](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L198) | [203](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L203) | [215](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L215) | [224](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L224) | [233](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L233) | [242](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L242) | [251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L251) | [264](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L264) | [268](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L268) | [270](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L270) | [277](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L277) | [321](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L321)

```solidity
File: src/launch/Airdrop.sol

48: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Only the BootstrapBallot can call Airdrop.authorizeWallet" );
49: require( ! claimingAllowed, "Cannot authorize after claiming is allowed" );
58: require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
59: require( ! claimingAllowed, "Claiming is already allowed" );
60: require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
76: require( claimingAllowed, "Claiming is not allowed yet" );
77: require( isAuthorized(msg.sender), "Wallet is not authorized for airdrop" );
78: require( ! claimed[msg.sender], "Wallet already claimed the airdrop" );
```
[48](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L48) | [49](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L49) | [58](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L58) | [59](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L59) | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L60) | [76](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L76) | [77](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L77) | [78](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol#L78)

```solidity
File: src/launch/BootstrapBallot.sol

36: require( ballotDuration > 0, "ballotDuration cannot be zero" );
50: require( ! hasVoted[msg.sender], "User already voted" );
54: require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );
71: require( ! ballotFinalized, "Ballot has already been finalized" );
72: require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
```
[36](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L36) | [50](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L50) | [54](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L54) | [71](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L71) | [72](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L72)

```solidity
File: src/launch/InitialDistribution.sol

52: require( msg.sender == address(bootstrapBallot), "InitialDistribution.distributionApproved can only be called from the BootstrapBallot contract" );
53: require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```
[52](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L52) | [53](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L53)

```solidity
File: src/pools/PoolsConfig.sol

47: require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
48: require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
136: require(address(pair.tokenA) != address(0) && address(pair.tokenB) != address(0), "This poolID does not exist");
```
[47](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L47) | [48](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L48) | [136](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol#L136)

```solidity
File: src/pools/PoolStats.sol

49: require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );
```
[49](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolStats.sol#L49)

```solidity
File: src/pools/Pools.sol

72: require( msg.sender == address(exchangeConfig.initialDistribution().bootstrapBallot()), "Pools.startExchangeApproved can only be called from the BootstrapBallot contract" );
83: require(block.timestamp <= deadline, "TX EXPIRED");
142: require( msg.sender == address(collateralAndLiquidity), "Pools.addLiquidity is only callable from the CollateralAndLiquidity contract" );
143: require( exchangeIsLive, "The exchange is not yet live" );
144: require( address(tokenA) != address(tokenB), "Cannot add liquidity for duplicate tokens" );
146: require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
147: require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );
158: require( addedLiquidity >= minLiquidityReceived, "Too little liquidity received" );
172: require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
173: require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
187: require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
193: require( (reclaimedA >= minReclaimedA) && (reclaimedB >= minReclaimedB), "Insufficient underlying tokens returned" );
207: require( amount > PoolUtils.DUST, "Deposit amount too small");
221: require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
222: require( amount > PoolUtils.DUST, "Withdraw amount too small");
243: require((reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves before swap");
271: require( (reserve0 >= PoolUtils.DUST) && (reserve1 >= PoolUtils.DUST), "Insufficient reserves after swap");
353: require( swapAmountOut >= minAmountOut, "Insufficient resulting token amount" );
371: require( userDeposits[swapTokenIn] >= swapAmountIn, "Insufficient deposited token balance of initial token" );
```
[72](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L72) | [83](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L83) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L142) | [143](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L143) | [144](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L144) | [146](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L146) | [147](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L147) | [158](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L158) | [172](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L172) | [173](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L173) | [187](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L187) | [193](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L193) | [207](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L207) | [221](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L221) | [222](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222) | [243](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L243) | [271](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L271) | [353](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L353) | [371](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L371)

```solidity
File: src/price_feed/PriceAggregator.sol

39: require( address(priceFeed1) == address(0), "setInitialFeeds() can only be called once" );
183: require (price != 0, "Invalid BTC price" );
195: require (price != 0, "Invalid ETH price" );
```
[39](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L39) | [183](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L183) | [195](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L195)

```solidity
File: src/rewards/Emissions.sol

42: require( msg.sender == address(exchangeConfig.upkeep()), "Emissions.performUpkeep is only callable from the Upkeep contract" );
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol#L42)

```solidity
File: src/rewards/RewardsEmitter.sol

63: require( poolsConfig.isWhitelisted( addedReward.poolID ), "Invalid pool" );
84: require( msg.sender == address(exchangeConfig.upkeep()), "RewardsEmitter.performUpkeep is only callable from the Upkeep contract" );
```
[63](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L63) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L84)

```solidity
File: src/rewards/SaltRewards.sol

60: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
108: require( msg.sender == address(exchangeConfig.initialDistribution()), "SaltRewards.sendInitialRewards is only callable from the InitialDistribution contract" );
119: require( msg.sender == address(exchangeConfig.upkeep()), "SaltRewards.performUpkeep is only callable from the Upkeep contract" );
120: require( poolIDs.length == profitsForPools.length, "Incompatible array lengths" );
```
[60](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L60) | [108](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L108) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L119) | [120](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/SaltRewards.sol#L120)

```solidity
File: src/stable/USDS.sol

42: require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
43: require( amount > 0, "Cannot mint zero USDS" );
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L42) | [43](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol#L43)

```solidity
File: src/stable/CollateralAndLiquidity.sol

83: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
84: require( collateralToWithdraw <= maxWithdrawableCollateral(msg.sender), "Excessive collateralToWithdraw" );
97: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
98: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
99: require( amountBorrowed <= maxBorrowableUSDS(msg.sender), "Excessive amountBorrowed" );
117: require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
118: require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
119: require( amountRepaid > 0, "Cannot repay zero amount" );
142: require( wallet != msg.sender, "Cannot liquidate self" );
145: require( canUserBeLiquidated(wallet), "User cannot be liquidated" );
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L83) | [84](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L84) | [97](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L97) | [98](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L98) | [99](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L99) | [117](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L117) | [118](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L118) | [119](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L119) | [142](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L142) | [145](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L145)

```solidity
File: src/stable/Liquidizer.sol

83: require( msg.sender == address(collateralAndLiquidity), "Liquidizer.incrementBurnableUSDS is only callable from the CollateralAndLiquidity contract" );
134: require( msg.sender == address(exchangeConfig.upkeep()), "Liquidizer.performUpkeep is only callable from the Upkeep contract" );
```
[83](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L83) | [134](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/Liquidizer.sol#L134)

```solidity
File: src/staking/Liquidity.sol

42: require(block.timestamp <= deadline, "TX EXPIRED");
85: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
124: require( userShareForPool(msg.sender, poolID) >= liquidityToWithdraw, "Cannot withdraw more than existing user share" );
148: require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be deposited via Liquidity.depositLiquidityAndIncreaseShare" );
159: require( PoolUtils._poolID( tokenA, tokenB ) != collateralPoolID, "Stablecoin collateral cannot be withdrawn via Liquidity.withdrawLiquidityAndClaim" );
```
[42](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L42) | [85](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L85) | [124](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L124) | [148](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L148) | [159](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L159)

```solidity
File: src/staking/StakingRewards.sol

59: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
60: require( increaseShareAmount != 0, "Cannot increase zero share" );
67: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
99: require( decreaseShareAmount != 0, "Cannot decrease zero share" );
102: require( decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share" );
107: require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
190: require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
```
[59](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L59) | [60](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L60) | [67](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L67) | [99](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L99) | [102](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L102) | [107](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L107) | [190](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L190)

```solidity
File: src/staking/Staking.sol

43: require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
62: require( userShareForPool(msg.sender, PoolUtils.STAKED_SALT) >= amountUnstaked, "Cannot unstake more than the amount staked" );
88: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be cancelled" );
89: require( block.timestamp < u.completionTime, "Unstakes that have already completed cannot be cancelled" );
90: require( msg.sender == u.wallet, "Sender is not the original staker" );
104: require( u.status == UnstakeState.PENDING, "Only PENDING unstakes can be claimed" );
105: require( block.timestamp >= u.completionTime, "Unstake has not completed yet" );
106: require( msg.sender == u.wallet, "Sender is not the original staker" );
132: require( msg.sender == address(exchangeConfig.airdrop()), "Staking.transferStakedSaltFromAirdropToUser is only callable from the Airdrop contract" );
153: require(end >= start, "Invalid range: end cannot be less than start");
157: require(userUnstakes.length > end, "Invalid range: end is out of bounds");
158: require(start < userUnstakes.length, "Invalid range: start is out of bounds");
204: require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
205: require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L43) | [62](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L62) | [88](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L88) | [89](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L89) | [90](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L90) | [104](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L104) | [105](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L105) | [106](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L106) | [132](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L132) | [153](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L153) | [157](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L157) | [158](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L158) | [204](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L204) | [205](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Staking.sol#L205)

```solidity
File: src/ManagedWallet.sol

44: require( msg.sender == mainWallet, "Only the current mainWallet can propose changes" );
45: require( _proposedMainWallet != address(0), "_proposedMainWallet cannot be the zero address" );
46: require( _proposedConfirmationWallet != address(0), "_proposedConfirmationWallet cannot be the zero address" );
49: require( proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet." );
61: require( msg.sender == confirmationWallet, "Invalid sender" );
76: require( msg.sender == proposedMainWallet, "Invalid sender" );
77: require( block.timestamp >= activeTimelock, "Timelock not yet completed" );
```
[44](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L44) | [45](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L45) | [46](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L46) | [49](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L49) | [61](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L61) | [76](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L76) | [77](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L77)

```solidity
File: src/AccessManager.sol

43: require( msg.sender == address(dao), "AccessManager.excludedCountriedUpdated only callable by the DAO" );
63: require( _verifyAccess(msg.sender, signature), "Incorrect AccessManager.grantAccess signatory" );
```
[43](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L43) | [63](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L63)

```solidity
File: src/SigningTools.sol

13: require( signature.length == 65, "Invalid signature length" );
```
[13](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L13)

```solidity
File: src/Upkeep.sol

97: require(msg.sender == address(this), "Only callable from within the same contract");
```
[97](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L97)

```solidity
File: src/ExchangeConfig.sol

51: require( address(dao) == address(0), "setContracts can only be called once" );
64: require( address(_accessManager) != address(0), "_accessManager cannot be address(0)" );
```
[51](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51) | [64](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L64)
</details>



