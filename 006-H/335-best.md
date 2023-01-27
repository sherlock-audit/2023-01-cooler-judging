cccz

medium

# Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom

## Summary
Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom
## Vulnerability Detail
Some tokens do not revert on failure, but instead return false (e.g. [ZRX](https://etherscan.io/address/0xe41d2489571d322189246dafa5ebde1f4699f498#code)).
https://github.com/d-xo/weird-erc20/#no-revert-on-failure
tranfser/transferfrom is directly used to send tokens in many places in the contract and the return value is not checked.
If the token send fails, it will cause a lot of serious problems.
For example, in the clear function, if debt token is ZRX, the lender can clear request without providing any debt token.
```solidity
    function clear (uint256 reqID) external returns (uint256 loanID) {
        Request storage req = requests[reqID];

        factory.newEvent(reqID, CoolerFactory.Events.Clear);

        if (!req.active) 
            revert Deactivated();
        else req.active = false;

        uint256 interest = interestFor(req.amount, req.interest, req.duration);
        uint256 collat = collateralFor(req.amount, req.loanToCollateral);
        uint256 expiration = block.timestamp + req.duration;

        loanID = loans.length;
        loans.push(
            Loan(req, req.amount + interest, collat, expiration, true, msg.sender)
        );
        debt.transferFrom(msg.sender, owner, req.amount);
    }
```
## Impact
If the token send fails, it will cause a lot of serious problems.
For example, in the clear function, if debt token is ZRX, the lender can clear request without providing any debt token.
## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L85-L86
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L122-L123
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L146-L147
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L179-L180
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L205-L206
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L102-L103
## Tool used

Manual Review

## Recommendation
Consider using safeTransfer/safeTransferFrom consistently.