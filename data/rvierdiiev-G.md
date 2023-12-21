## G-01
## Description
`setPosMode` function loops through all borrowed pools and checks [if it is allowed to borrow in new mode and repay in old pool](https://github.com/code-423n4/2023-12-initcapital/blob/main/contracts/core/InitCore.sol#L203-L204) inside the loop. However it's possible to do that only 1 time outside the loop in case if pools length is not 0. This will save gas and will reduce those 2 lines calculation for n times to 1.

