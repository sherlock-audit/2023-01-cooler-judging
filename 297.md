Zarf

high

# Protocol assumes debt token has 18 decimals

## Summary

Arithmetic operations are performed with the assumption that the debt token always has 18 decimals.

For example, when computing the interest on the debt or calculating the collateral amount, `1e18` is hardcoded in the formula. 

## Vulnerability Detail

If we assume the debt token is using <18 decimals, the borrower will have to overpay in collateral and the interest will be lower than expected.

If we assume the debt token is using >18 decimals, the borrower will have to put up less collateral and the interest will be higher than expected.

(Assuming the `loanToCollateral` remains unchanged and is calculated for a debt token with 18 decimals)

```solidity
function collateralFor(uint256 amount, uint256 loanToCollateral) public pure returns (uint256) {
    return amount * decimals / loanToCollateral;
}

function interestFor(uint256 amount, uint256 rate, uint256 duration) public pure returns (uint256) {
    uint256 interest = rate * duration / 365 days;
    return amount * interest / decimals;
}
```

## Impact

Depending on the amount of decimals used by the debt token, this might be in favor to the lender/borrower and ‘steal’ funds from the other party. 

## Code Snippet

[https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L54](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L54)

[https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L236-L248](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L236-L248)

## Tool used

Manual Review

## Recommendation

Don’t use a hardcoded amount of decimals but fetch the amount of decimals used by the debt token in the constructor. For example:

```solidity
constructor (address o, ERC20 c, ERC20 d) {
    owner = o;
    collateral = c;
    debt = d;
    decimals = debt.decimals();
    factory = CoolerFactory(msg.sender);
}
```