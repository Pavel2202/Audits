# Curves report by Timenov

## Summary

|               | Issue                                | Instances |
| ------------- | :----------------------------------- | :-------: |
| [H-01](#h-01) | Subject can block token selling. |     1     |
| [H-02](#h-02) | No access control in FeeSplitter::setCurves. |     1     |

### [H-01]<a name="h-01"></a> Subject can block token selling.

#### Impact

The way that the `sell` mechanism works is the following:

- User calls `sellCurvesToken()`
- Transferring the fees through `_transferFees()`

The problem lies in `_transferFees()`. In case of selling, the protocol:

1. sends eth to the `msg.sender`
2. sends `subjectFee` to the subject
3. sends `referralFee` to the referral(if any)
4. calls `onBalanceChange` and `addFees` functions of `feeRedistributor`.

A malicious subject or referral can take benefit of that implementation. He can either block all users of selling or he can allow only himself to sell his tokens. This can be achieved if the `subject` or `referral` himself is a smart contract with no `receive()` or the `receive()` only allows the malicious user to sell.

```solidity
            (bool success2,) = curvesTokenSubject.call{value: subjectFee}("");
            if (!success2) revert CannotSendFunds();
```

```solidity
            (bool success3,) = referralDefined
                ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
                : (true, bytes(""));
            if (!success3) revert CannotSendFunds();
```

#### Proof of Concept
We will take a look only at the `referralFeeDestination` attack.

1. Create smart contract with no `receive()`

```solidity
pragma solidity 0.8.7;

contract BadReceiver {}
```

2. Modify one of the test cases in `friend-tech.ts`

```diff
it("Should be able to buy and sell tokens by a friend", async () => {
  await testContract.buyCurvesToken(owner.address, 1);
  const initialPrice = await testContract.getBuyPrice(owner.address, 2);
  await testContract.connect(friend).buyCurvesToken(owner.address, 2, { value: initialPrice });

  const friendBalance = await testContract.curvesTokenBalance(owner.address, friend.address);
  expect(friendBalance).to.equal(2);

  @audit add these lines
+      const BAD_RECEIVER_FACTORY = await ethers.getContractFactory("BadReceiver", owner);
+      const badReceiver = await BAD_RECEIVER_FACTORY.deploy();
+      await testContract.connect(owner).setReferralFeeDestination(owner.address, badReceiver.address);
+      const newReferralAddress = await testContract.referralFeeDestination(owner.address);
+      expect(newReferralAddress).to.equal(badReceiver.address);

  @audit this will fail
  const tx = await testContract.connect(friend).sellCurvesToken(owner.address, 1);
  const rec = await tx.wait();
  //@ts-ignore
  const trade = rec.events[0];
  const ICurves = Curves__factory.createInterface();
  //@ts-ignore
  const parsedEvent = ICurves.parseLog(trade) as TradeEvent;

  const friendBalance2 = await testContract.curvesTokenBalance(owner.address, friend.address);
  expect(friendBalance2).to.equal(1);
  expect(parsedEvent.args.ethAmount).to.equal(250000000000000);
  expect(parsedEvent.args.protocolEthAmount).to.equal(0);
  expect(parsedEvent.args.subjectEthAmount).to.equal(0);
});
```

Full file can be seen in this [Gist](https://gist.github.com/Pavel2202/61d1c2db0bb623cddb02d78fb4887dbc)

3. Test fails with correct error

```solidity
Error: VM Exception while processing transaction: reverted with custom error 'CannotSendFunds()'
```

#### Lines of code

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218-L261

#### Tools Used

Manual Review, Hardhat

#### Recommended Mitigation Steps

I know this is a fork of `friend.tech` and the selling functionality there is implemented the same. The problem is that, here there are 3 more calls(`referral.call()`, `feeRedistributor.onBalanceChange()` and `feeRedistributor.addFees()`). This opens up the opportunity for more points of failure. Consider implementing a `pull over push` strategy for the fess.

### [H-02]<a name="h-02"></a> No access control in FeeSplitter::setCurves.

#### Impact

`FeeSplitter` is used to manage the distribution of fees. The storage variable `curves` is a reference to the Curves smart contract. It is used to retrieve the user's balance and get the total supply. If we take a look at the function `setCurves`, we can see that there we set the `curves` variable.

```solidity
  function setCurves(Curves curves_) public {
      curves = curves_;
  }
```

The problem here is that anyone can call this function as there is no access control. This function alone can cause huge problems across the whole protocol.

#### Proof of Concept
Added the following test case to `fee-token-holders.ts`:

```js
  it("Anyone can change curves address", async () => {
    let curvesAddress = await testContract.curves();
    expect(curvesAddress).to.equal(mockCurves.address);

    await testContract.connect(friend).setCurves(friend.address);
    curvesAddress = await testContract.curves();
    expect(curvesAddress).to.equal(friend.address)
  });
```

Test results:

```js
Fee token holders test
  Fee Splitter
    ✔ Anyone can change curves address
```

Here is a [Gist](https://gist.github.com/Pavel2202/407eda86db1cce857aab5e5c65eccf02) with the updated test file.

#### Lines of code

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L35-L37

#### Tools Used

Manual Review, Hardhat

#### Recommended Mitigation Steps

I think that the `onlyOwner` modifier would suit this function the most(after [this](https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md#h-01) is fixed of course). So after the changes, the `setCurves` function should look like this:

```solidity
  function setCurves(Curves curves_) public onlyOwner {
      curves = curves_;
  }
```