ck

medium

# Plutus integration will require contract whitelisting otherwise all transactions will fail.

## Summary

The Plutus GLP Depositor contract has checks that will need to be considered before the `PLVGLPController` can operate.

## Vulnerability Detail

The Plutus GLP Depositor contract has an eligibility check as follows:

```solidity
function _isEligibleSender() private view {
    if (
      msg.sender != tx.origin && whitelist.isWhitelisted(msg.sender) == false && partners[msg.sender].isActive == false
    ) revert UNAUTHORIZED();
  }
```

This is checked for all deposits and redemptions. The proxy nature of accounts in sentiment means ` msg.sender != tx.origin` would fail. Whitelisting of the relevant contract will therefore need to be whitelisted by Plutus after deployment.

## Impact

The eligibility requirements are an important element that Sentiment needs to consider both in the short term and long term.  In the short term, Sentiment should start the process of whitelisting in advance to prevent unwarranted delays of scheduled launches. In the long term, the risk of being locked out of the Plutus protocol should also be considered.

## Code Snippet

https://arbiscan.io/address/0x13F0D29b5B83654A200E4540066713d50547606E#code#F1#L170

## Tool used

Manual Review

## Recommendation

Ensure eligibility requirements will be accounted for as the contracts are deployed.