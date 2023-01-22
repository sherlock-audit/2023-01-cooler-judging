0xadrii

high

# Malicious Cooler owner can steal deposited collateral from borrower

## Summary
Anybody can request a loan for a given Cooler contract, but only the Cooler owner can rescind it.
## Vulnerability Detail
The `request` function in Cooler allows any arbitrary user to create a loan request, asking for an amount to borrow and depositing a computed amount of tokens as collateral. Loan requests can then be cancelled via the `rescind` function, which will set the loan request as unactive and return the deposited collateral tokens. The problem is that  only the Cooler contract owner will be able to call `rescind` to  cancel a loan, so the owner will always get the collateral deposited tokens in request, even if the borrow requester was not the owner. 
Scenario:
1. Alice creates a Cooler contract with the Factory, so she becomes the owner
2. Because there is no access control restrictions in `request`, Bob  requests for a loan, depositing an amount of tokens as collateral.
3. Alice notices this, and calls `rescind` (she is the only one able to do so), thus obtaining the collateral deposited by Bob in the previous step and cancelling Bob's loan request.
## Impact
High
## Code Snippet
`Cooler.sol`:
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L74-L86
```solidity
 function request (
        uint256 amount,
        uint256 interest,
        uint256 loanToCollateral,
        uint256 duration
    ) external returns (uint256 reqID) {
       //@audit anyone can call this function and request a loan
        reqID = requests.length;
        factory.newEvent(reqID, CoolerFactory.Events.Request);
        requests.push(
            Request(amount, interest, loanToCollateral, duration, true)
        );
        collateral.transferFrom(msg.sender, address(this), collateralFor(amount, loanToCollateral));
    }
```

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L90-L103
```solidity
function rescind (uint256 reqID) external {
// @audit access is restricted to only Cooler owner
  if (msg.sender != owner) 
      revert OnlyApproved();
  
  factory.newEvent(reqID, CoolerFactory.Events.Rescind);
  
  Request storage req = requests[reqID];
  
  if (!req.active)
      revert Deactivated();
  
  req.active = false;
  // @audit collateral will directly be transferred to Cooler owner
  collateral.transfer(owner, collateralFor(req.amount, req.loanToCollateral));
}
```
## Tool used
Manual Review

## Recommendation
It seems that the intended behavior for Cooler is to only allow for the creator of the contract to create loan requests (in [Cooler documentation](https://ag0.gitbook.io/cooler-loans/origination), the borrower always seems to be the Cooler contract owner).
If that is the case, the `request` function should also restrict its access to only the Cooler owner:
```solidity
 function request (
        uint256 amount,
        uint256 interest,
        uint256 loanToCollateral,
        uint256 duration
    ) external returns (uint256 reqID) {
       //@audit this should be added to restrict access
       if (msg.sender != owner) 
               revert OnlyApproved();
        reqID = requests.length;
        factory.newEvent(reqID, CoolerFactory.Events.Request);
        requests.push(
            Request(amount, interest, loanToCollateral, duration, true)
        );
        collateral.transferFrom(msg.sender, address(this), collateralFor(amount, loanToCollateral));
    }
```

On the other hand, if the intended behavior is to allow any arbitrary user to create loan requests, the `rescind` function should change its access control to only allow the loan requester to rescind the loan request. To track the requester address, the address of the requester could be stored in the Request struct.
