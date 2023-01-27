kiki_dev

high

# Attacker can request a loan of a significant amount without providing any collateral

## Summary
It is possible to create a request with the `collateral` being 0 and the `amount` being non-zero.

## Vulnerability Detail
If a user request a loan with `loantoCollateral` being greater than `amount * decimals` then the function `collateralFor()` will return 0. Because  the return value for `collateralFor()` is what is sent to the collateral contract in this instance the attacker will send 0. 

The issue is that `request()` will still create a request. If a lender were to look at the submtited request all they would see is the `amount` requested, the `intererest` being set, the `loanToCollateral `, and the `duration`. The request provides no information about how much collateral exist. 

Unless a user looks at the code finds the formula for `collateralfor()`, and calculates it themselves they will have no idea that there is zero collateral available. If a user were to accept that request the Attacker can keep the loan and the users wouldn't get anything when the loan defaults. 

If you run this test you will see that a user can make a request for a non-zero amount and submit zero collateral. Once an victim accepts this request, which is likely because from their point of view this is a good deal. the attacker will have effectively stolen the victims money.

    function test_zeroCollateral() public {
        // possible to return 0 collateral
        uint cooler0 = gOHM.balanceOf(address(cooler));

        gOHM.approve(address(cooler), 1e18);

        uint256 amount_ = 100; // 100 * ie18
        uint256 interest_ = 5e17; //50%
        uint256 ltc_ = 101e18;
        uint256 duration_ = 365 days;

        uint256 reqId = cooler.request(amount_, interest_, ltc_, duration_); // makes request
        assertTrue(gOHM.balanceOf(address(cooler)) == cooler0); // balance of collateral does not change after request
    }

## Impact
Any owner can make a request with 0 collateral submitted and steal a lenders asset. 
## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L102
## Tool used

Manual Review

## Recommendation
Put a check in `collaterFor()` that ensures return value is not 0.