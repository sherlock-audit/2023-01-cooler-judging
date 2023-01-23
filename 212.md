HonorLt

medium

# Asynchronous toggle

## Summary
Due to the asynchronous nature of blockchains, it is not advised to use toggle functions.

## Vulnerability Detail
You have a function ```toggleRoll``` in ```Cooler``` contract:
```solidity
    function toggleRoll(uint256 loanID) external returns (bool) {
        Loan storage loan = loans[loanID];

        if (msg.sender != loan.lender)
            revert OnlyApproved();

        loan.rollable = !loan.rollable;
        return loan.rollable;
    }
```
With blockchains, you cannot be sure when your tx will be confirmed, for example, you might invoke the toggle function with too low gas price and later send another tx with a higher price but at this time your first tx might be confirmed.

## Impact
It is possible that several toggle invocations might overlap and change the state in an unexpected order.

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L182-L193

## Tool used

Manual Review

## Recommendation
I recommend having separate enable/disable roll functions, similar to pause/unpause in the famous OZ contract.
