# Lybra Finance report by Timenov

## Summary

[[H-01](#H-01)] The `onlyRole` modifier has no check.
[[H-02](#H-02)] The `checkRole` modifier has no check.

### [H-01]<a name="H-01"></a> The `onlyRole` modifier has no check.

_There is 1 instance of this issue._

#### Impact

The modifier `onlyRole` has no `require` nor `revert` statement. This modifier is used in 4 functions. Having no check would mean that this modifier will always be bypassed and would result in everyone having the ability to call the `initToken`, `setMintVault`, `setMintVaultMaxSupply` and `setBadCollateralRatio` functions.

#### Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L85-L88

#### Tools used

VSCode

#### Recommended Mitigation Steps

Add `require` or `revert` statement that would revert if certain condition is not met.

### [H-02]<a name="H-02"></a> The `checkRole` modifier has no check.

_There is 1 instance of this issue._

#### Impact

The modifier `checkRole` has no `require` nor `revert` statement. This modifier is used in 13 functions. Having no check would mean that this modifier will always be bypassed and would result in everyone having the ability to call the `setProtocolRewardsPool`, `setEUSDMiningIncentives`, `setvaultBurnPaused`, `setPremiumTradingEnabled`, `setvaultMintPaused`, `setRedemptionFee`, `setSafeCollateralRatio`, `setBorrowApy`, `setKeeperRatio`, `setTokenMiner`, `setMaxStableRatio`, `setFlashloanFee` and `setProtocolRewardsToken` functions.

#### Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L90-L93

#### Tools used

VSCode

#### Recommended Mitigation Steps

Add `require` or `revert` statement that would revert if certain condition is not met.
