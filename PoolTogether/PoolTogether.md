# PoolTogether report by Timenov

## Summary

|               | Issue                                | Instances |
| ------------- | :----------------------------------- | :-------: |
| [M-01](#m-01) | permit will not work on some tokens. |     1     |

### [M-01]<a name="m-01"></a> permit will not work on some tokens.

#### Impact

The `depositWithPermit` function will not work as expected if the `_asset` does not support the `permit` functionality.

#### Proof of Concept
Some tokens(for example [WETH](https://solidity-by-example.org/hacks/weth-permit/) and [stETH](https://etherscan.io/address/0x47ebab13b806773ec2a2d16873e2df770d130b50#code#F10#L50)) do not have a `permit` function and others(for example [DAI](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f#code#L171)) utilizes a `permit` function that deviates from the reference implementation.

This means that the permit will execute, but the allowance will not be correct, resulting in unexpected behaviour. The tokens above are widely used so it is likely for them to be the `_asset` of the vault. Also there are more tokens that have the same issues.

#### Lines of code

https://github.com/code-423n4/2024-03-pooltogether/blob/480d58b9e8611c13587f28811864aea138a0021a/pt-v5-vault/src/PrizeVault.sol#L540

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Consider adding a validation after the `permit` to check if the allowance is correct and revert with a message if not.

```solidity
require(_asset.allowance(_owner, address(this)) >= _assets, "Allowance with permit failed.");
```