rvierdiiev

medium

# ClearingHouse doesn't have ability to approve loan

## Summary
ClearingHouse doesn't have ability to approve loan, so it doesn't support all function inside Cooler and operator of ClearingHouse will not be able to sell his Loan to someone else.
## Vulnerability Detail
Cooler contract has `approve` function which allows Loan's owner to approve transfer of loan to another address. Once it's approves, then another account can call `transfer` function in order to accept loan and change it's owner.

ClearingHouse is created to work with different coollers for the operator. It has different functions that integrate functionality of Cooler contract.
However ClearingHouse doesn't implement approve function. So it's not possbile for operator to approve transfer of its loan. As result, selling of loans to someone else can be blocked.
## Impact
There is no ability to approve loan to another account. Selling of loans is blocked.
## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L212-L229
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/aux/ClearingHouse.sol#L7
## Tool used

Manual Review

## Recommendation
Add ability to approve loans to another account and also to accept transfer when loan is sold to ClearingHouse.