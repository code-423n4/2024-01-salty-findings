For a one man dev team this salty codebase is pretty damn rock solid!

I tried hard to DoS the liquidation functionality via at least two different methods, and was unable to.
Tried repeatedly calling all 3 price sources in a `while` loop from a separate attack transaction but to no avail.
Also tried a repeated deposit-withdraw approach which also didnt stop the liquidation.

I mean, the main/only? complaint I have about the codebase is the placement of the curly braces... Otherwise it's very intelligently implemented.
Very unconventional approach/method of implementation too in some aspects.
Modular approach was wise.

Even when still using the outdated method `latestRoundData` in the chainlink oracle, the dev still manages to implement it in such a way as to try mitigate most of the issues that come along with `latestRoundData`.

It seems clear that the codebase was implemented with careful handling of different decimal values and conversion to 18 decimals wherever necessary.

The codebase makes smart use of try-catch blocks in critical areas to properly handle potential cases of errors and reverts that could otherwise pose unexpected issues to the normal functioning of the protocol, which can mean the difference between a bug/vulnerability and "all good".

The codebase makes good use of double `if` statements instead of `&&`, to help save on gas.

All in all it seems that the most serious vulnerabilities/bugs in this solid codebase(if any exist?) would be found mostly via invariant fuzzing, as long as all the business logic has been implemented 100% without logic errors.

Another smart implementation in the PoolMath library, where instead of doing this:
`(zapAmountA / zapAmountB) < (reserveA / reserveB)`
It is implemented like this:
`(zapAmountA * reserveB) < (reserveA * zapAmountB)`
eliminating the risks which come with division.

Extremely well implemented codebase, clearly lots of thought and planning went into it.
I did not have time to deep-dive everything but from my bug hunting efforts I think the pricing mechanism could probably be improved in terms of reliability... It's already pretty good/reliable, but I foresee that it's possible for things to go wrong. Removing the hardcoded limit of 30 days so that DAO could implement a new pricefeed immediately/soon if ever necessary, would be a wise approach. Still use the 30 days timelock limit, but have an emergency option in place that would make it possible to bypass the 30 day timelock.



### Time spent:
84 hours