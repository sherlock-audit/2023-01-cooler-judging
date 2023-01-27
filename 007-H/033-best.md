0x52

high

# Fully repaying a loan will result in debt payment being lost

## Summary

When a `loan` is fully repaid the `loan` storage is deleted. Since `loan` is a `storage` reference to the loan, `loan.lender` will return `address(0)` after the `loan` has been deleted. This will result in the `debt` being transferred to `address(0)` instead of the lender. Some ERC20 tokens will revert when being sent to `address(0)` but a large number will simply be sent there and lost forever.

## Vulnerability Detail

    function repay (uint256 loanID, uint256 repaid) external {
        Loan storage loan = loans[loanID];

        if (block.timestamp > loan.expiry) 
            revert Default();
        
        uint256 decollateralized = loan.collateral * repaid / loan.amount;

        if (repaid == loan.amount) delete loans[loanID];
        else {
            loan.amount -= repaid;
            loan.collateral -= decollateralized;
        }

        debt.transferFrom(msg.sender, loan.lender, repaid);
        collateral.transfer(owner, decollateralized);
    }

In `Cooler#repay` the loan storage associated with the loanID being repaid is deleted. `loan` is a storage reference so when `loans[loanID]` is deleted so is `loan`. The result is that `loan.lender` is now `address(0)` and the loan payment will be sent there instead.

## Impact

Lender's funds are sent to `address(0)`

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L108-L124

## Tool used

Manual Review

## Recommendation

Send collateral/debt then delete:

    -   if (repaid == loan.amount) delete loans[loanID];
    +   if (repaid == loan.amount) {
    +       debt.transferFrom(msg.sender, loan.lender, loan.amount);
    +       collateral.transfer(owner, loan.collateral);
    +       delete loans[loanID];
    +       return;
    +   }