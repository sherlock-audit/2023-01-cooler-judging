IllIllI

high

# Loans can be rolled an unlimited number of times

## Summary

Loans can be rolled an unlimited number of times, without letting the lender decide if has been done too many times already


## Vulnerability Detail

The lender is expected to be able to toggle whether a loan can be rolled or not, but once it's enabled, there is no way to prevent the borrower from rolling an unlimited number of times in the same transaction or in quick succession.


## Impact

If the lender is giving an interest-free loan and assumes that allowing a roll will only extend the term by one, they'll potentially be forced to wait until the end of the universe if the borrower chooses to roll an excessive number of times.

If the borrower is using a quickly-depreciating collateral, the lender may be happy to allow one a one-term extension, but will lose money if the term is rolled multiple times and the borrower defaults thereafter.

The initial value of `loan.rollable` is always `true`, so unless the lender calls `toggleRoll()` in the same transaction that they call `clear()`, a determined attacker will be able to roll as many times as they wish.


## Code Snippet

As long as the borrower is willing to pay the interest up front, they can call `roll()` any number of times, extending the duration of the total loan to however long they wish:
```solidity
// File: src/Cooler.sol : Cooler.roll()   #1

129        function roll (uint256 loanID) external {
130            Loan storage loan = loans[loanID];
131            Request memory req = loan.request;
132    
133            if (block.timestamp > loan.expiry) 
134                revert Default();
135    
136            if (!loan.rollable)
137                revert NotRollable();
138    
139            uint256 newCollateral = collateralFor(loan.amount, req.loanToCollateral) - loan.collateral;
140            uint256 newDebt = interestFor(loan.amount, req.interest, req.duration);
141    
142            loan.amount += newDebt;
143            loan.expiry += req.duration;
144            loan.collateral += newCollateral;
145            
146            collateral.transferFrom(msg.sender, address(this), newCollateral);
147:       }
```
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L129-L147

[`toggleRoll()`](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L185-L193) can't be used to stop rolls if they're all done in a single transaction.


## Tool used

Manual Review


## Recommendation

Have a variable controlling the number of rolls the lender is allowing, and or only allow a roll if the current `block.timestamp` is within one `req.duration` of the current `loan.expiry`

