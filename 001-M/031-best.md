ck

medium

# Chainlink's `latestRoundData` might return stale or incorrect results

## Summary

`GLPOracle::getEthPrice` checks for staleness by only using `updatedAt`. However this can still lead to stale prices  if a stale price was carried over to the current round. 

## Vulnerability Detail

The staleness check of `getEthPrice` does not account for stale prices being carried over to the current round.

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

According to Chainlink documentation: "If `answeredInRound` is less than `roundId`, the answer is being carried over. If `answeredInRound` is equal to `roundId`, then the answer is fresh". 

It should therefore not be assumed that `updatedAt` alone always indicates a fresh price.

## Impact

This could lead to stale prices.

## Code Snippet

https://github.com/sherlock-audit/2023-01-sentiment/blob/main/oracle/src/gmx/GLPOracle.sol#L47-L58

## Tool used

Manual Review

## Recommendation

Update the function to include the additional `roundID` check.

```solidity
function getEthPrice() internal view returns (uint) {
        (uint80 roundID, int256 answer, , uint256 updatedAt, uint80 answeredInRound) = ethUsdPriceFeed.latestRoundData();
        require(answeredInRound >= roundID, "Stale price");
        if (block.timestamp - updatedAt >= 86400)
            revert Errors.StalePrice(address(0), address(ethUsdPriceFeed));

        if (answer <= 0)
            revert Errors.NegativePrice(address(0), address(ethUsdPriceFeed));
        return uint(answer);
}
```