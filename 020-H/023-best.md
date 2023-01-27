bin2chen

high

# Lender force Loan become default

## Summary
in ```repay()``` directly transfer the debt token to Lender, but did not consider that Lender can not accept the token (in contract blacklist), resulting in repay() always revert, and finally the Loan can only expire, Loan be default

## Vulnerability Detail
The only way for the borrower to get the collateral token back is to repay the amount owed via repay(). Currently in the repay() method transfers the debt token directly to the Lender.
This has a problem:
 if the Lender is blacklisted by the debt token now, the debtToken.transferFrom() method will fail and the repay() method will always fail and finally the Loan will default.
Example:
Assume collateral token = ETH,debt token = USDC, owner = alice
1.alice call request() to loan 2000 usdc , duration = 1 mon
2.bob call clear(): loanID =1
3.bob transfer loan[1].lender = jack by Cooler.approve/transfer  
  Note: jack has been in USDC's blacklist for some reason before
or bob in USDC's blacklist for some reason now, it doesn't need transfer 'lender')
4.Sometime before the expiration date, alice call repay(id=1) , it will always revert, Because usdc.transfer(jack) will revert
5.after 1 mon, loan[1] default, jack call defaulted() get collateral token

```solidity
    function repay (uint256 loanID, uint256 repaid) external {
        Loan storage loan = loans[loanID];
...
        debt.transferFrom(msg.sender, loan.lender, repaid);   //***<------- lender in debt token's blocklist will revert , example :debt = usdc
        collateral.transfer(owner, decollateralized);
    }
```

## Impact

Lender forced Loan become default for get collateral token, owner lost collateral token

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L122

## Tool used

Manual Review

## Recommendation

Instead of transferring the debt token directly, put the debt token into the Cooler.sol and set like: withdrawBalance[lender]+=amount, and provide the method withdraw() for lender to get debtToken back