
The comment in the function `_bisectionSearch` says that:

```
	// Assuming that the profit function is unimodal (has only one peak), the two profit calculations can show us which half of the range the maximum profit is in (where to keep looking).
```

But I think this profit function is monotonically increasing. The profit function is composed of three parts, where the output of the former is the input of the latter. If these three functions are monotonically increasing, then the entire profit function is also increasing on [0, $\infty$).

We have:
`amountOut = (reservesA1 * bestArbAmountIn) / (reservesA0 + bestArbAmountIn);`

so we can get this funciton: `y = (reservesA1 * x) / (reservesA0 + x)` where $x > 0,  reservesA0 > 100, reservesA1 > 100$.

Let $x_1 > x_0 >= 0$, so $y_1 - y_0 = (reservesA1 * x_1) / (reservesA0 + x_1) - (reservesA1 * x_0) / (reservesA0 + x_0)$
$= reservesA1 * (reservesA0*x_1 + x_1*x_0 - reservesA0*x_0 - x_1*x_0) / (reservesA0 + x_0)*(reservesA0 + x_1)$
$= reservesA1 * reservesA0 * (x_1 - x_0) / (reservesA0 + x_0)*(reservesA0 + x_1)$

Because $x_1 > x_0$, so $x_1 - x_0 > 0$, which means $y_1 > y_0$ when $x_1 > x_0$, this function is monotonically increasing.

Therefore, I think there is no need to use binary search. The maximum return is always obtained at the time of maximum input.

### Time spent:
30 hours