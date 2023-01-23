HonorLt

medium

# Loan is rollable by default

## Summary
Making the loan rollable by default gives an unfair early advantage to the borrowers.

## Vulnerability Detail
When clearing a new loan, the flag of ```rollable``` is set to true by default:
```solidity
    loans.push(
        Loan(req, req.amount + interest, collat, expiration, true, msg.sender)
    );
```
This means a borrower can extend the loan anytime before the expiry:
```solidity
    function roll (uint256 loanID) external {
        Loan storage loan = loans[loanID];
        Request memory req = loan.request;

        if (block.timestamp > loan.expiry) 
            revert Default();

        if (!loan.rollable)
            revert NotRollable();
```
If the lenders do not intend to allow rollable loans, they should separately toggle the status to prevent that:
```solidity
    function toggleRoll(uint256 loanID) external returns (bool) {
        ...
        loan.rollable = !loan.rollable;
        ...
    }
```

I believe it gives an unfair advantage to the borrower because they can re-roll the loan before the lender's transaction forbids this action.

## Impact
Lenders who do not want the loans to be used more than once, have to bundle their transactions. Otherwise, it is possible that someone might roll their loan, especially if the capital requirements are not huge because anyone can roll any loan.

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L177

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L191

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L126-L147

## Tool used

Manual Review

## Recommendation
I believe ```rollable``` should be set to false by default or at least add an extra function parameter to determine the initial value of this status.