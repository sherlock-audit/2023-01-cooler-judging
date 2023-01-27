HonorLt

medium

# Disincentivized early repayments

## Summary
```repay``` does not account for the elapsed duration making it impractical to do it earlier than just before the expiry.

## Vulnerability Detail
When repaying the loan, the collateralized amount is calculated based on the total ```loan.amount```:
```solidity
    function repay (uint256 loanID, uint256 repaid) external {
        ...
        uint256 decollateralized = loan.collateral * repaid / loan.amount;
```
The ```loan.amount``` already includes the whole annual interest (```req.amount + interest```):
```solidity
  loans.push(
    Loan(req, req.amount + interest, collat, expiration, true, msg.sender)
  );
```
Users can partially or fully repay the loan anytime. However, this makes it impractical to do it sooner than later because the borrower still has to pay the full interest.

I believe it should account for the elapsed period compared to the total duration and not charge for the whole duration.

## Impact

Because it is a fixed-ratio loan, users risk no liquidation and the protocol provides no incentives for the borrowers to repay earlier than just right before the expiry. The utilization and flow of assets are not as optimal as possible.

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L114

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L177

## Tool used

Manual Review

## Recommendation
Consider accounting for the elapsed duration when repaying the loan, and probably add a min duration to prevent instant open and close spam.