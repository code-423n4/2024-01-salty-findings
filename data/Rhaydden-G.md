Heres the link to the affected lines: https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/pools/PoolMath.sol#L210-L228

In the PoolMath.sol, We can optimize gas by using Short Circuit Evaluation. The && and || operators use short circuit evaluation. This means that if the first operand in a logical AND (&&) or OR (||) operation determines the outcome, the second operand is not evaluated. This can save gas if the first operand makes the outcome clear. We can apply this principle in the _determineZapSwapAmount function:

```// Original Code
if ( zapAmountA * reserveB > reserveA * zapAmountB )
   (swapAmountA, swapAmountB) = (_zapSwapAmount( reserveA, reserveB, zapAmountA, zapAmountB ), 0);

if ( zapAmountA * reserveB < reserveA * zapAmountB )
   (swapAmountA, swapAmountB) = (0, _zapSwapAmount( reserveB, reserveA, zapAmountB, zapAmountA ));

// Optimized Code
if ( zapAmountA * reserveB > reserveA * zapAmountB )
   (swapAmountA, swapAmountB) = (_zapSwapAmount( reserveA, reserveB, zapAmountA, zapAmountB ), 0);
else if ( zapAmountA * reserveB < reserveA * zapAmountB )
   (swapAmountA, swapAmountB) = (0, _zapSwapAmount( reserveB, reserveA, zapAmountB, zapAmountA ));```





