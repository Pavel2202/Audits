# Lybra Finance report by Timenov

## Summary

| |Issue|Instances|
|-|:-|:-:|
| [H-01](#h-01) | The `onlyRole` modifier has no check. | 1 |
| [H-02](#h-02) | The `checkRole` modifier has no check. | 1 |

### [H-01]<a name="h-01"></a> The `onlyRole` modifier has no check.

_There is 1 instance of this issue._

#### Impact

The modifier `onlyRole` has no `require` nor `revert` statement. This modifier is used in 4 functions. Having no check would mean that this modifier will always be bypassed and would result in everyone having the ability to call the `initToken`, `setMintVault`, `setMintVaultMaxSupply`, and `setBadCollateralRatio` functions.

#### Lines of code

[LybraConfigurator.sol#L85-L88](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L85-L88)

#### Tools used

VSCode

#### Recommended Mitigation Steps

Add `require` or
