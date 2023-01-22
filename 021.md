0x52

high

# Malicious lender can roll the loan for the borrower to force them to pay more interest or cause them to default

## Summary

Each time a loan is rolled the amount of debt the must be repaid grows larger. A malicious seller can use this to their advantage to increase the loan amount and make it harder for them to pay the loan back. For the seller it is a win-win scenario. When they roll the loan for the borrow, they supply some collateral to back the higher loan amount but they also stand to make more money from the interest payments. This results in one of the following scenarios:

1) The borrow repays their rolled loan and the seller effectively sells their collateral to the borrow and nets the higher amount of interest

2) The borrower can't come up with more money to pay the loan so they default and the seller gets their collateral back and the borrower's which is presumably worth more than the value of the loan.

## Vulnerability Detail

See summary.

## Impact

Either way borrower losses funds

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L129-L147

## Tool used

Manual Review

## Recommendation

Only the owner (borrower) should be allowed to roll a loan