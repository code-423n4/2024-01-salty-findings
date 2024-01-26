this voting system uses Ethereum addresses, and can users create multiple addresses for extra votes. So, the current way isn't stopping that trick.

	// Ensures that voters can only vote once
	mapping(address=>bool) public hasVoted;