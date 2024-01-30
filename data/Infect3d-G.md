# [G-01] - The `_maximumMSB` function could be rewritten in a way more efficient way

The `_maximumMSB` function could be rewritten in a way more efficient way :

```diff
File: src\pools\PoolMath.sol
130: 	// Determine the maximum msb across the given values
131: 	function _maximumMSB( uint256 r0, uint256 r1, uint256 z0, uint256 z1 ) internal pure returns (uint256 msb)
132: 		{
133: +		msb = _mostSignificantBit(r0 | r1 | z0 | z1);
133: -		msb = _mostSignificantBit(r0);
134: -
135: -		uint256 m = _mostSignificantBit(r1);
136: -		if ( m > msb )
137: -			msb = m;
138: -
139: -		m = _mostSignificantBit(z0);
140: -		if ( m > msb )
141: -			msb = m;
142: -
143: -		m = _mostSignificantBit(z1);
144: -		if ( m > msb )
145: -			msb = m;
146: 		}
```