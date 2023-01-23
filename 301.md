stent

medium

# Token transfers leave reentrancy attacks a possibility

## Summary

If one of the ERC20 tokens (debt or collateral) was malicious it is possible to steal funds with a reentrancy attack.

## Vulnerability Detail

There are a few places where attacks can be made. Here is 1 example:
```solidity
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
```
Suppose the collateral token is malicious. When the repay function is called (presumably by the owner) then debt tokens are transferred to the lender before the collateralized ones and so the transfer function in the collateral token could call the repay function again which would send another batch of tokens to the lender. The callback can be written in such a way that only 1 batch of collateral tokens is sent to the owner. Example:
```solidity
function transfer(address to, uint256 amount) public {
  if (firstTime) {
    firstTime = false;
    cooler.repay(loanID, stealAmount);
  } else {
    super.transfer(to, amount);
  }
}
```

## Impact

Potential loss of funds. It is up to the owner to do their due diligence and check that the ERC20 contracts being used for debt and collateral do not have reentrancy attacks embedded in them.

## Code Snippet

https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L123

## Tool used

Manual Review

## Recommendation

Create a boolean defense flag:

```solidity
    bool times;
    function repay (uint256 loanID, uint256 repaid) external {
        times +=1;
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
        
        if (times != 1) revert;
        times = 0;
    }
```