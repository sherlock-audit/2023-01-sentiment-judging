Bahurum

high

# Use of [`fsGLP`] instead of [`sGLP`] in `PLVGLPController` and `RewardRouterV2Controller`

## Summary
In `PLVGLPController` and `RewardRouterV2Controllerl` [`sGLP`](https://arbiscan.io/address/0x5402B5F40310bDED796c7D0F3FF6683f5C0cFfdf) is used instead of [`fsGLP`](https://arbiscan.io/token/0x1aDDD80E6039594eE970E5872D247bf0414C8903) as token. This means that when depositing or redeeming `fsGLP` will not be added to the account collateral and the margin for the account will be underestimated, which can cause early liquidation of the account.

## Vulnerability Detail
- When an account calls `mintAndStakeGlpETH` on `RewardRouterV2`, the router sends to it [`fsGLP`](https://arbiscan.io/token/0x1aDDD80E6039594eE970E5872D247bf0414C8903), as in [this tx](https://arbiscan.io/tx/0x8ea5c7c29c29b4c146198d514e731184628ac9c05414e779ea1e1ab6fcb09e72). The contract adds [`sGLP`](https://arbiscan.io/address/0x5402B5F40310bDED796c7D0F3FF6683f5C0cFfdf) instead to `tokensIn`.

- When an account calls `redeem` on PLV GLP Vault, the router sends to it [`fsGLP`](https://arbiscan.io/token/0x1aDDD80E6039594eE970E5872D247bf0414C8903), as in [this tx](https://arbiscan.io/tx/0x9609c2bd2ddc060b5a6ebaac3ac77b6e3e5ad16bfb5a9f25e7bc3debf8b62518). The contract adds [`sGLP`](https://arbiscan.io/address/0x5402B5F40310bDED796c7D0F3FF6683f5C0cFfdf) instead to `tokensIn`. 

In both cases, the tokens received are ignored and not added to the account's collateral. The margin will be unexpectedly low and the account can face early liquidation.


## Impact
The collateral calculated will be lower than the actual value of tokens in the account. This is unexpected by the owner of the account, and he can be liquidated earlier than expected.

## Code Snippet

https://github.com/sherlock-audit/2023-01-sentiment/blob/main/controller-55/src/gmx/RewardRouterV2Controller.sol#L28-L29

https://github.com/sherlock-audit/2023-01-sentiment/blob/main/controller-55/src/plutus/PLVGLPController.sol#L28-L29

## Tool used

Manual Review

## Recommendation
In both contracts, use the `fsGLP` token at address `0x1aDDD80E6039594eE970E5872D247bf0414C8903` instead of the token `sGLP` at the address `0x5402B5F40310bDED796c7D0F3FF6683f5C0cFfdf`.