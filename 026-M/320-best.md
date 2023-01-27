hansfriese

medium

# `Cooler.roll()` wouldn't work as expected when `newCollateral = 0`.



## Summary
`Cooler.roll()` is used to increase the loan duration by transferring the additional collateral.

But there will be some problems when `newCollateral = 0`.

## Vulnerability Detail
```solidity
    function roll (uint256 loanID) external {
        Loan storage loan = loans[loanID];
        Request memory req = loan.request;

        if (block.timestamp > loan.expiry) 
            revert Default();

        if (!loan.rollable)
            revert NotRollable();

        uint256 newCollateral = collateralFor(loan.amount, req.loanToCollateral) - loan.collateral;
        uint256 newDebt = interestFor(loan.amount, req.interest, req.duration);

        loan.amount += newDebt;
        loan.expiry += req.duration;
        loan.collateral += newCollateral;
        
        collateral.transferFrom(msg.sender, address(this), newCollateral); //@audit 0 amount
    }
```

In `roll()`, it transfers the `newCollateral` amount of collateral to the contract.

After the borrower repaid most of the debts, `loan.amount` might be very small and `newCollateral` for the original interest might be 0 because of the rounding issue.

Then as we can see from this [one](https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers), some tokens might revert for 0 amount and `roll()` wouldn't work as expected.

## Impact
There will be 2 impacts.

1. When the borrower tries to extend the loan using `roll()`, it will revert with the weird tokens when `newCollateral = 0`.
2. After the borrower noticed he couldn't repay anymore(so the lender will default the loan), the borrower can call `roll()` again when `newCollateral = 0`.
In this case, the borrower doesn't lose anything but the lender must wait for `req.duration` again to default the loan.

## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L146

## Tool used
Manual Review

## Recommendation
I think we should handle it differently when `newCollateral = 0`.

According to impact 2, I think it would be good to revert when `newCollateral = 0`.