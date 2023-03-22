# RabbitHole Quest Protocol report by Timenov

## Summary
G-01 Using `x += y` costs more gas than using `x = x + y`.
G-02 Using `mapping(uint256 => bool)` costs more than using BitMap.
H-01 The `onlyMinter` modifier has no check.

### [G-01] Using `x += y` costs more gas than using `x = x + y`.
*There is 1 instances of this issue.*

```solidity
File: contracts/Quest.sol

115: redeemedTokens += redeemableTokenCount;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

### [G-02] Using `mapping(uint256 => bool)` costs more than using BitMap.

We can use BitMap instead of `mapping(uint256 => bool)`.

To import it use:

```solidity
import {BitMaps} from "@openzeppelin/contracts/utils/structs/BitMaps.sol";
```

To use it add:

```solidity
using BitMaps for BitMaps.BitMap;
```

To declare it use:
```solidity
BitMaps.BitMap private bitmap;
```

*There is 1 instances of this issue.*

```solidity
File: contracts/Quest.sol

24: mapping(uint256 => bool) private claimedList;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

### [H-01] The `onlyMinter` modifier has no check.

## Impact
The modifier `onlyMinter` used in `RabbitHoleReceipt.sol` and `RabbitHoleTickets.sol` has no `require` nor `revert` statement. This modifier is used in 3 functions. Having no check would mean that this modifier will always be bypassed and would result in everyone having the ability to call the `mint` and `mintBatch` functions.

## Proof of Concept
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

## Recommended Mitigation Steps
Add `require(msg.sender == minterAddress, "Error message.");` or `if (msg.sender == minterAddress) revert CustomError();`