neumo

medium

# GLPOracle's getEthPrice interval to assume a stale price is too big

## Summary
The interval assumed by `getEthPrice` to consider the ETH price stale is 24 hours. Although this value is the heartbeat of the ETH / USD feed in Arbitrum, the deviation threshold is 0.05%, so updates in the price of the feed usually happen every few seconds beacause of the high volatility of ETH. This means `GLPOracle` should consider the price stale way before the time of the heartbeat, otherwise if the aggregator is not working correctly, the price of ETH from the feed could be deviated by a considerable amount and the functions relying on `getETHPrice` would return a wrong value.

## Vulnerability Detail
Function `getEthPrice`, defined in `GLPOracle` looks like this:
```solidity
function getEthPrice() internal view returns (uint) {
	(, int answer,, uint updatedAt,) =
		ethUsdPriceFeed.latestRoundData();


	if (block.timestamp - updatedAt >= 86400)
		revert Errors.StalePrice(address(0), address(ethUsdPriceFeed));


	if (answer <= 0)
		revert Errors.NegativePrice(address(0), address(ethUsdPriceFeed));


	return uint(answer);
}
```

It reverts when the price returned by the aggregator was updated more than one day (86,400 seconds) before the call, as it assumes price is stale.

This number is too big (although it's the heartbeat for the Arbitrum ETH / USD feed) and could lead to returning wrong prices for a long period before the 24 hours after last update ends and the call starts reverting.

The [Arbitrum ETH / USD feed](https://arbiscan.io/address/0x639fe6ab55c921f74e7fac1ee960c0b6293ba612) has updates of the price every few seconds, because the deviation threshold in Arbitrum is 0.05%. Also, the volatility of ETH makes it almost impossible that the price does not deviate more that 0.05% in a 24 hour period. So if the aggregator malfunctions for whatever reason and does not update the price, the impact of considering the price valid for 24 hours could be high.

From the Chainlink documentation:
> If your application detects that the reported answer is not updated within the heartbeat or within time limits that you determine are acceptable for your application, pause operation or switch to an alternate operation mode while identifying the cause of the delay.

I think the time limit acceptable (although it is subjective) for Sentiment to avoid impactful mispricing of assets should be lowered. The Ethereum feed has a 1 hour heartbeat with a higher deviation rate (0.5%), I recommend using a value of 3,600 seconds, like the heartbeat of Ethereum, to consider the price stale, and avoid the impact of mispricing sentiment assets if the Aggregator fails.


This value of 86,400 seconds for the ETH / USD feed is used in other contracts of the protocol:
* WSTETHOracle
* ChainlinkOracle

And should be updated too.

## Impact
Medium

## Code Snippet
https://github.com/sherlock-audit/2023-01-sentiment/blob/main/oracle/src/gmx/GLPOracle.sol#L47-L58

## Tool used
Manual review


## Recommendation
Lower the time to consider the price of the ETH / USD feed stale to a value of 3,600.

