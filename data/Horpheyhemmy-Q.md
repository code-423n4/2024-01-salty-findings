## L-1: Lack Of Events For Some Critical Parameters 


## SUMMARY

Function ``Parameters::_executeParameterChange`` sets critical parameters, But it has no event emitted , Could lead to architectural error trying to track these changes off-chain.

## VULNERABILITY DETAILS

In ``Parameters::_executeParameterChange`` , critical Parameter are set for the pool,dao,staking,price,reward and stable config But no event is emitted.

It would be difficult tracking those changes off-chain. 

## IMPACT

Both User and Issuers would not be aware of these changes

## CODE SNIPPET

https://github.com/code-423n4/2024-01-salty/blob/main/src%2Fdao%2FParameters.sol#L57

## TOOL USED

Manual

##  RECOMMENDATION 

Emit an Event in ``Parameters::_executeParameterChange`` to report the configuration changes.


