# 2023-07-beedle
My CodeHawks submission for Beedle Audit: Oracle free peer to peer perpetual lending

# Beedle - Oracle free perpetual lending - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. The lender can change the interest rate at any time by resetting the pool without a delay](#H-01)


- ## Gas Optimizations / Informationals
    - ### [G-01. `_burn` and `_afterTokenTransfer` functions in `Beedle.sol` are unnecessary](#G-01)
    - ### [G-02. Wrong comment in `setPool` function](#G-02)
    - ### [G-03. No check if `p.loanToken` and `p.collateralToken` are contracts when updating the pool](#G-03)
    - ### [G-04. Use `require` instead of `if` with `revert`](#G-04)

# <a id='contest-summary'></a>Contest Summary

### Sponsor: BeedleFi

### Dates: Jul 24th, 2023 - Aug 7th, 2023

[See more contest details here](https://www.codehawks.com/contests/clkbo1fa20009jr08nyyf9wbx)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 1
   - Medium: 0
   - Low: 0
  - Gas/Info: 4

# High Risk Findings

## <a id='H-01'></a>H-01. The lender can change the interest rate at any time by resetting the pool without a delay            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L130

https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L221

## Summary
`setPool` and `updateInterestRate` functions in `Lender.sol` contract allow the lender to change the interest rate to up to 1000% at any time without a delay, this also includes frontrunning the borrowers' transactions. 

## Vulnerability Details

## Impact
The arbitrary change of interest rates may be unexpected for borrowers and can result in an inability to repay the loan on time. 

## Tools Used
Manual Review

## Recommendations
Consider adding a delay (for instance: 1 day) for the interest rate update to go live in both the `setPool` and `updateInterestRate` functions.

		




# Gas Optimizations / Informationals

## <a id='G/I-01'></a>G/I-01. `_burn` and `_afterTokenTransfer` functions in `Beedle.sol` are unnecessary            



## Summary
Internal functions `_burn`and `_afterTokenTransfer` are unnecessary, as they are present in `ERC20.sol` already. This is a tautology.

## Recommendations
Remove functions `_burn`and `_afterTokenTransfer` from `Beedle.sol`.

## <a id='G/I-02'></a>G/I-02. Wrong comment in `setPool` function            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L141-L142

https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L148

## Vulnerability Details and Recommendations

In the `Lender.sol` contract's `setPool` function, replace:
```
        // check if they already have a pool balance
        poolId = getPoolId(p.lender, p.loanToken, p.collateralToken);

        ...

        uint256 currentBalance = pools[poolId].poolBalance;
```
with:
```
        // get pool id
        poolId = getPoolId(p.lender, p.loanToken, p.collateralToken);

        ...

        // check if they already have a pool balance
        uint256 currentBalance = pools[poolId].poolBalance;

```

## <a id='G/I-03'></a>G/I-03. No check if `p.loanToken` and `p.collateralToken` are contracts when updating the pool            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L132

## Summary
`setPool` function in `Lender.sol` contract does not check if pool parameters `p.loanToken` and `p.collateralToken` are contracts. 

## Tools Used
Manual Review

## Recommendations
Use OpenZeppelin [isContract(address account)](https://docs.openzeppelin.com/contracts/2.x/api/utils#Address-isContract-address-) function to check if the input address is a contract 

Example:
```
function setPool(Pool calldata p) public returns (bytes32 poolId) {
    ...
    require(isContract(p.loanToken) && isContract(p.collateralToken), "Token is not a contract");
    ...
}
```

## <a id='G/I-04'></a>G/I-04. Use `require` instead of `if` with `revert`            



## Summary and Recommendations
Even though there's no significant difference in gas cost, it's generally recommended to use `require` for input validation and pre-condition checks, and `if` with `revert` for more complex logic.

This is applicable across the whole codebase

## Tools Used
Manual Review
