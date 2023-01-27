0xhacksmithh

medium

# ```requests``` Array get crowded, as ```rescind()``` did not deleting borrower's deactivated request

## Summary
```request[]``` get more and more crowed during course of time.

## Vulnerability Detail
```rescind``` does not removing deactivated requests in rescind function from requests[], which makes requests[] more and more crowded in further future.

```solidity
 function rescind (uint256 reqID) external { 
        if (msg.sender != owner) 
            revert OnlyApproved();

        factory.newEvent(reqID, CoolerFactory.Events.Rescind);

        Request storage req = requests[reqID]; 

        if (!req.active)
            revert Deactivated();
        
        req.active = false;
        collateral.transfer(owner, collateralFor(req.amount, req.loanToCollateral));  
    }
```

## Impact
Due more crowd, it will cost more gas to set variables(request) in ```request[]```

## Code Snippet
https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L90-L103
## Tool used

Manual Review

## Recommendation
Should remove deactivated requests in rescind function from requests[], like other functionality do in same code base,
For example,
defaulted() function delete loan from loans[] after settlement of that particular loan.

Use these type of code in that rescind() function  