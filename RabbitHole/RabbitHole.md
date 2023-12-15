# RabbitHole Quest Protocol report by Timenov

## Summary
H-01 The `onlyMinter` modifier has no check.

### [H-01] The `onlyMinter` modifier has no check.

#### Impact
The modifier `onlyMinter` used in `RabbitHoleReceipt.sol` and `RabbitHoleTickets.sol` has no `require` nor `revert` statement. This modifier is used in 3 functions. Having no check would mean that this modifier will always be bypassed and would result in everyone having the ability to call the `mint` and `mintBatch` functions.

#### Proof of Concept
```solidity
File: contracts/RabbitHoleReceipt.sol

58:     modifier onlyMinter() {
        msg.sender == minterAddress;
        _;
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58

```solidity
File: contracts/RabbitHoleReceipt.sol

47:     modifier onlyMinter() {
        msg.sender == minterAddress;
        _;
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L47

#### Recommended Mitigation Steps
Add `require(msg.sender == minterAddress, "Error message.");` or `if (msg.sender == minterAddress) revert CustomError();`