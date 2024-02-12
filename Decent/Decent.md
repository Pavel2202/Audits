# Decent report by Timenov

## Summary

|               | Issue                                | Instances |
| ------------- | :----------------------------------- | :-------: |
| [H-01](#h-01) | Anyone can change router in DcntEth. |     1     |

### [H-01]<a name="h-01"></a> Anyone can change router in DcntEth.

#### Impact

There is no access modifier in `DcntEth::setRouter`. Therefore anyone can call this function and change the `router`. This will bypass the modifier `onlyRouter` when `mint()` and `burn()` are called.

#### Proof of Concept
In this PoC, I will show how the attack can happen:

1. Create folder `audit` in the `test` folder.

2. Create file called `DcntEthTest.t.sol`.

3. Copy from this [Gist](https://gist.github.com/Pavel2202/49d1fb748ead9f918dfb3714627590a0) and paste into the file.

4. Run `forge test --match-test test_change_router` in the terminal.

5. Test passes:

#### Lines of code

https://github.com/decentxyz/decent-bridge/blob/7f90fd4489551b69c20d11eeecb17a3f564afb18/src/DcntEth.sol#L20-L22

#### Tools Used

Manual Review, Foundry

#### Recommended Mitigation Steps

Add `onlyOwner` modifier to `setRouter` function:

```diff
-    function setRouter(address _router) public {
+    function setRouter(address _router) public onlyOwner {
         router = _router;
     }
```
