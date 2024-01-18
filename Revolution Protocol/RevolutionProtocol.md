# Revolution Protocol report by Timenov

## Summary
| |Issue|Instances|
|-|:-|:-:|
| [M-01](#m-01) | `settleCurrentAndCreateNewAuction` can be DoS. | 1 |

### [M-01]<a name="m-01"></a> `settleCurrentAndCreateNewAuction` can be DoS.

#### Lines of code

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L152-L155


#### Vulnerability details

##### Impact
The `AuctionHouse::settleCurrentAndCreateNewAuction` function is used to first settle the current auction and create a new one and can be called by anyone. Lets take a look at `_settleAuction()`. Here we send the highest bidder the `VerbToken`, pay the creators(ETH and ERC20) and pay the owner. The max amount of creators an art can have is `100`. If malicious user adds art with 100 creators(smart contracts with unbounded loops) that will consume all the gas when receiving ETH and his art wins, the following check in `_createAuction`: `require(gasleft() >= MIN_TOKEN_MINT_GAS_THRESHOLD, "Insufficient gas for creating auction");
` might fail. If somehow this doesn't fail, the user that called `settleCurrentAndCreateNewAuction` will end up paying a lot of gas, which may lead to less trust in the contract.

##### Proof of Concept
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L394

https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L311

##### Tools Used
Manual Review

##### Recommended Mitigation Steps
Use `pull over push` method when sending the rewards to the creators(ETH and ERC20).


##### Assessed type

DoS