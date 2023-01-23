rvierdiiev

medium

# Cooler doesn't take collateral from owner to cover interests

## Summary
Cooler doesn't take collateral from owner to cover interests.
## Vulnerability Detail
When new request is created then loan amount is [collateralized by owner](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L85).
But as you can see user pays collateral for loan amount only.
But in the end of loan user should also pay interests for lender. And in case if user will not repay it, then lender will receive collateral for loan amount and interests will not be compensated.

Suppose that owner take a 1000 usdc loan and backs it with 1000 dai. So rate in this example is 1:1. Lender is not increasing collateral rate he is going to earn by interests paid by owner of Cooler. Interests were aggreed by 2 sides to pay 15 usdc at the end.
However borrower didn't repay his debt and as result lender received 1000 dai, but interests were not compensated.

I think that borrower should collateralize interests as well. So in this case he should provide 1015 dai as collateral at the request creation time.
## Impact
Loss of interests for lender.
## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L85
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L205
## Tool used

Manual Review

## Recommendation
Borrower should back interests with collateral when creates request.