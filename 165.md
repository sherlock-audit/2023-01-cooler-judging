0x4non

medium

# Multiply before Dividing: Precision issues on `interestFor` function

## Summary
Performing multiplication before division is generally better to avoid loss of precision because Solidity integer division might truncate

## Vulnerability Detail
Since you are loosing precision because you divide first an multiply later you could end with a smaller number than you supposed to.
Please read this article: [Solidity Design Patterns: Multiply before Dividing
](https://medium.com/@soliditydeveloper.com/solidity-design-patterns-multiply-before-dividing-407980646f7)

## Impact
A bad actor could take advantage of this issue to take a loan with lower interest that is supposed to taking a high loan with a small duration and a small rate.

## Code Snippet
This is the vulnerable function [Cooler.sol/#L245-L248](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol/#L245-L248)
```solidity
    function interestFor(uint256 amount, uint256 rate, uint256 duration) public pure returns (uint256) {
        uint256 interest = rate * duration / 365 days;
        return amount * interest / decimals;
    }
```

And in this lines is used;
[Cooler.sol#L140](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L140)
[Cooler.sol#L171](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol#L171)

## Tool used
Manual Review

## Recommendation
Multiply first, and then divide, change this function [Cooler.sol/#L245-L248](https://github.com/sherlock-audit/2023-01-cooler/blob/main/src/Cooler.sol/#L245-L248)
```solidity
    function interestFor(uint256 amount, uint256 rate, uint256 duration) public pure returns (uint256) {
        uint256 interest = rate * duration / 365 days;
        return amount * interest / decimals;
    }
```

To this:
```solidity
    function interestFor(uint256 amount, uint256 rate, uint256 duration) public pure returns (uint256) {
        uint256 numerator = rate * duration * amount;
        uint256 denominator = 365 days * decimals;
        return  numerator / denominator;
    }
```