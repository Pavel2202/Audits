# MorpheusAI report by Timenov

## Summary

|               | Issue                                                     | Instances |
| ------------- | :-------------------------------------------------------- | :-------: |
| [L-01](#l-01) | L1Sender does not implement `supportsInterface` function. |     1     |

### [L-01]<a name="l-01"></a> L1Sender does not implement `supportsInterface` function.

#### Impact

The contracts `MOR`, `L2TokenReceiver` and `L1Sender` inherit `ERC165`. This standard is used to publish and detect what interfaces a smart contract implements. Read [this](https://eips.ethereum.org/EIPS/eip-165) for more. If we take a look at `MOR` and `L2TokenReceiver`, we can see that they implement this standard the correct way:

```solidity
// MOR.sol

    function supportsInterface(bytes4 interfaceId_) external pure returns (bool) {
        return
            interfaceId_ == type(IMOR).interfaceId ||
            interfaceId_ == type(IERC20).interfaceId ||
            interfaceId_ == type(IERC165).interfaceId;
    }
```

```solidity
// L2TokenReceiver.sol

    function supportsInterface(bytes4 interfaceId_) external pure returns (bool) {
        return interfaceId_ == type(IL2TokenReceiver).interfaceId || interfaceId_ == type(IERC165).interfaceId;
    }
```

But there is no such function in `L1Sender.sol`.

#### Vulnerability Details

Having no `supportsInterface` function, means that the `ERC165` standard is not implement correctly. This contract implements `IL1Sender` and `ERC165`. Lets say someone wants to check if this contract implements the `IL1Sender` interface. The lack of `supportsInterface` function, means that the return value will be `false`, which is wrong.

#### Lines of code

https://github.com/Cyfrin/2024-01-Morpheus/blob/76898177fbedcbbf4b78b513d9fa151bbf3388de/contracts/L1Sender.sol#L16

#### Proof of Concept

Do the following changes to L1Sender.sol:

- Import the IERC165 interface at the top of the file:

`import {ERC165, IERC165} from "@openzeppelin/contracts/utils/introspection/ERC165.sol";`

- Create the following pure functions:

```solidity
    function getL1SenderInterface() external pure returns (bytes4) {
        return type(IL1Sender).interfaceId;
    }

    function getIERC165Interface() external pure returns (bytes4) {
        return type(IERC165).interfaceId;
    }
```

Full file can be found in this [Gist](https://gist.github.com/Pavel2202/52f466cc10ffcc96a9ec3a9e529e9485).

Now add the following test cases to `L1Sender.test.ts`

```js
describe("supportsInterface", () => {
  it("should support l1Sender interface", async () => {
    const l1SenderInterfaceId = await l1Sender.getL1SenderInterface();
    expect(await l1Sender.supportsInterface(l1SenderInterfaceId)).to.be.true;
  });

  it("should support IERC165 interface", async () => {
    const erc165InterfaceId = await l1Sender.getIERC165Interface();
    expect(await l1Sender.supportsInterface(erc165InterfaceId)).to.be.true;
  });
});
```

Run `npx hardhat test`.

```solidity
  L1Sender
    supportsInterface
      1) should support l1Sender interface
      ✔ should support IERC165 interface
```

As we can see, the first test fails(which is the `IL1Sender`) and the second(`IERC165`) passes. The second passes because in `ERC165` the `supportsInterface` functions is this:

```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
        return interfaceId == type(IERC165).interfaceId;
    }
```

#### Tools Used

Manual Review, Hardhat

#### Recommended Mitigation Steps

Make sure to implement correctly the `supportsInterface` function:

```solidity
    function supportsInterface(bytes4 interfaceId_) public override pure returns (bool) {
        return interfaceId_ == type(IL1Sender).interfaceId || interfaceId_ == type(IERC165).interfaceId;
    }
```

Note that we add a check for `IERC165` here as well, because this function is `override`.

Now `run npx hardhat test` again:

```solidity
  L1Sender
    supportsInterface
      ✔ should support l1Sender interface
      ✔ should support IERC165 interface
```
