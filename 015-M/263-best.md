berndartmueller

medium

# Repaying loans with small amounts of debt tokens can lead to underflowing in the `roll` function

## Summary

Due to precision issues when repaying a loan with small amounts of debt tokens, the `loan.amount` can be reduced whereas the `loan.collateral` remains unchanged. This can lead to underflowing in the `roll` function.

## Vulnerability Detail

The `decollateralized` calculation in the `repay` function rounds down to zero if the `repaid` amount is small enough. This allows iteratively repaying a loan with very small amounts of debt tokens without reducing the collateral.

The consequence is that the `roll` function can revert due to underflowing the `newCollateral` calculation once the `loan.collateral` is greater than `collateralFor(loan.amount, req.loanToCollateral)` (`loan.amount` is reduced by repaying the loan)

As any ERC-20 tokens with different decimals can be used, this precision issue is amplified if the decimals of the collateral and debt tokens differ greatly.

## Impact

The `roll` function can revert due to underflowing the `newCollateral` calculation if the `repay` function is (iteratively) called with small amounts of debt tokens.

## Code Snippet

[Cooler.sol#L114](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L114)

```solidity
function repay (uint256 loanID, uint256 repaid) external {
    Loan storage loan = loans[loanID];

    if (block.timestamp > loan.expiry)
        revert Default();

    uint256 decollateralized = loan.collateral * repaid / loan.amount; // @audit-info (10e18 * 10) / 1_000e18 = 0 (rounds down due to imprecision)

    if (repaid == loan.amount) delete loans[loanID];
    else {
        loan.amount -= repaid;
        loan.collateral -= decollateralized;
    }

    debt.transferFrom(msg.sender, loan.lender, repaid);
    collateral.transfer(owner, decollateralized);
}
```

[Cooler.sol#L139](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L139)

Calculating `newCollateral` in L139 can potentially revert due to underflowing if `loan.collateral` is greater than the required collateral (`collateralFor(loan.amount, req.loanToCollateral) `).

A malicious user can use the imprecision issue in the `repay` function in L114 to repay small amounts of debt tokens (`loan.collateral * repaid` < `loan.amount`), which leads to no reduction of loan collateral, whereas the `loan.amount` is reduced.

This will prevent the `roll` function from being called.

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

    collateral.transferFrom(msg.sender, address(this), newCollateral);
}
```

## Tool used

Manual Review

## Recommendation

Consider preventing the loan from being repaid if the amount of returned collateral tokens is zero (i.e., `decollateralized == 0`).
