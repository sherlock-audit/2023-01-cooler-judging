IllIllI

medium

# Dust amounts can cause payments to fail, leading to default

## Summary

Dust amounts can cause payments to fail, leading to default


## Vulnerability Detail

In order for a loan to close, the exact right number of wei of the debt token must be sent to match the remaining loan amount. If more is sent, the balance underflows, reverting the transaction.


## Impact

An attacker can send dust amounts right before a loan is due, front-running any payments also destined for the final block before default. If the attacker's transaction goes in first, the borrower will be unable to pay back the loan before default, and will lose thier remaining collateral. This may be the whole loan amount.


## Code Snippet

If the repayment amount isn't exactly the remaining loan amount, and instead is more (due to the dust payment), the subtraction marked below will underflow, reverting the payment:
```solidity
// File: src/Cooler.sol : Cooler.repay()   #1

108        function repay (uint256 loanID, uint256 repaid) external {
109            Loan storage loan = loans[loanID];
110    
111            if (block.timestamp > loan.expiry) 
112                revert Default();
113            
114            uint256 decollateralized = loan.collateral * repaid / loan.amount;
115    
116           if (repaid == loan.amount) delete loans[loanID];
117           else {
118 @>             loan.amount -= repaid;
119                loan.collateral -= decollateralized;
120            }
121    
122            debt.transferFrom(msg.sender, loan.lender, repaid);
123            collateral.transfer(owner, decollateralized);
124:       }
```
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L108-L124


## Tool used

Manual Review


## Recommendation

Only collect and subtract the minimum of the current loan balance, and the amount specified in the `repaid` variable

