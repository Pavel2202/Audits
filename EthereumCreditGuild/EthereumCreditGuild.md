# EthereumCreditGuild report by Timenov

## Summary

|               | Issue                    | Instances |
| ------------- | :----------------------- | :-------: |
| [M-01](#m-01) | Wrong term maximum debt. |     1     |

### [M-01]<a name="m-01"></a> Wrong term maximum debt.

#### Impact

The last comment in `debtCeiling` functions states: `return min(creditMinterBuffer, hardCap, debtCeiling)`. However there is an edge case that does not return the smallest of the three parameters.

#### Proof of Concept

Lets assume the following values:

`creditMinterBuffer = 2`,
`_debtCeiling = 3`,
`_hardCap = 1`

The first check in line 324 is `if (creditMinterBuffer < _debtCeiling)`. We pass this and the function returns `2`.

#### Lines of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L323-L330

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Add additional checks in the if statements.

```solidity
        if (creditMinterBuffer < _debtCeiling && creditMinterBuffer < _hardCap) {
            return creditMinterBuffer;
        }
        if (_hardCap < _debtCeiling && _hardCap < creditMinterBuffer) {
            return _hardCap;
        }
        return _debtCeiling;
```
