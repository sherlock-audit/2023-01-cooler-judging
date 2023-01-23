imare

medium

# protocol will not work correctly with fee-on-transfer/rebasing token

## Summary
Protocol is incompatible with fee-on-transfer/rebasing token

## Vulnerability Detail

In the different functions of the protocol, we use the amount user passed in for accounting, but we not check the actual amount we received.

In `request` function
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L85

`rescind` function

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L102

`repain` function

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L122

`roll` also uses values suplied by the user stored inside the `Loan` struct

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L139-L146

## Impact

## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L85

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L102

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L139-L146

## Tool used

Manual Review

## Recommendation
We recommend the project use before and after balance check to check how many token we actually receive instead of assume we received the exact token amount from user's input.