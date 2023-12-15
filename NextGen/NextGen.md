# NextGen report by Timenov

## Summary
M-01 Attacker can DoS auction by winning it.

### [M-01] Attacker can DoS auction by winning it.

#### Impact
Lets imagine the following scenario. Alice enters the auction with `1 ether`. After that Bob enters as well with `2 ether`. Eve(a malicious user) decides to DoS the auction. To do that she creates a smart contract that does not implement the `IERC721Receiver` interface and has no `onERC721Received()` function, so the contract will not be able to receive any NFTs. Admin calls `claimAuction()`, but the function reverts because it can't send the NFT to Eve's contract. Therefore the other users(Alice and Bob) will not get their money back.

#### Lines of code

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104-L120

#### Tools Used
Manual Review

#### Recommended Mitigation Steps
Refactor the `claimAuction()` function to only change the state of the current auction with the `tokenId`. And when the auction is in finalized state, allow the winner to get it through `claimNFT()` with `WinnerOrAdminRequired` modifier.