# Teller report by Timenov

## Summary

|               | Issue                                | Instances |
| ------------- | :----------------------------------- | :-------: |
| [H-01](#h-01) | Lender may not be able to close loan or get back lending token. |     1     |
| [H-02](#h-02) | Lender can not close loan with recipient. |     1     |
| [H-03](#h-03) | Borrower can keep loan. |     1     |

### [H-01]<a name="h-01"></a> Lender may not be able to close loan or get back lending token.

The `TellerV2` smart contract offers lenders to claim a NFT by calling `claimLoanNFT`. This will set the `bid.lender` to the constant variable `USING_LENDER_MANAGER`. This is done to ensure that the lender will only be able to claim 1 NFT per bid. Because of that `getLoanLender` function is added to check who the real owner of the NFT is(the real lender). However that whole logic introduces problems that can cause lender to not be able to close a loan or get his lending tokens.

#### Vulnerability Detail
Lets split the issues and look at them separately:

##### Not being able to close a loan

A lender can close a loan as long as the bid is accepted. This can be done by calling lenderCloseLoan or lenderCloseLoan.

```solidity
    function lenderCloseLoan(uint256 _bidId)
        external
        acceptedLoan(_bidId, "lenderClaimCollateral")
    {
        Bid storage bid = bids[_bidId];
        address _collateralRecipient = bid.lender;

        _lenderCloseLoanWithRecipient(_bidId, _collateralRecipient);
    }
```

```solidity
    function lenderCloseLoanWithRecipient(
        uint256 _bidId,
        address _collateralRecipient
    ) external {
        _lenderCloseLoanWithRecipient(_bidId, _collateralRecipient);
    }
```

As can be seen, both functions call internal function `_lenderCloseLoanWithRecipient`.

```solidity
    function _lenderCloseLoanWithRecipient(
        uint256 _bidId,
        address _collateralRecipient
    ) internal acceptedLoan(_bidId, "lenderClaimCollateral") {
        require(isLoanDefaulted(_bidId), "Loan must be defaulted.");

        Bid storage bid = bids[_bidId];
        bid.state = BidState.CLOSED;

        address sender = _msgSenderForMarket(bid.marketplaceId);
        require(sender == bid.lender, "only lender can close loan");

        /*


          address collateralManagerForBid = address(_getCollateralManagerForBid(_bidId)); 

          if( collateralManagerForBid == address(collateralManagerV2) ){
             ICollateralManagerV2(collateralManagerForBid).lenderClaimCollateral(_bidId,_collateralRecipient);
          }else{
             require( _collateralRecipient == address(bid.lender));
             ICollateralManager(collateralManagerForBid).lenderClaimCollateral(_bidId );
          }
          
          */

        collateralManager.lenderClaimCollateral(_bidId);

        emit LoanClosed(_bidId);
    }
```

Notice this validation `require(sender == bid.lender, "only lender can close loan");`. If a lender claims loan NFT, the bid.lender will be `USING_LENDER_MANAGER`. This means that this check will fail and lender will not be able to close the loan. Also the same validation can be seen in `setRepaymentListenerForBid`.

##### Not being able to get lending tokens

Lender got his loan NFT and now his bid is either PAID or LIQUIDATED. This will call the internal function _sendOrEscrowFunds:

```solidity
    function _sendOrEscrowFunds(uint256 _bidId, Payment memory _payment)
        internal
    {
        Bid storage bid = bids[_bidId];
        address lender = getLoanLender(_bidId);

        uint256 _paymentAmount = _payment.principal + _payment.interest;

        // some code
    }
```

This time the lender is determined by calling `getLoanLender` function:

```solidity
    function getLoanLender(uint256 _bidId)
        public
        view
        returns (address lender_)
    {
        lender_ = bids[_bidId].lender;

        if (lender_ == address(USING_LENDER_MANAGER)) {
            return lenderManager.ownerOf(_bidId);
        }

        //this is left in for backwards compatibility only
        if (lender_ == address(lenderManager)) {
            return lenderManager.ownerOf(_bidId);
        }
    }
```

If the bid.lender is `USING_LENDER_MANAGER`, then the owner of the NFT is the lender. However this may not be true as the lender can lose(transfer to another account or get stolen), or sell(without knowing it's purpose) . This means that the new owner of the NFT will get the lending tokens as well.

#### Impact
Lenders not being able to close a loan or get back lending tokens.

#### Code Snippet
https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L724-L732

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L738-L743

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L745-L755

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L901-L905

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L1173-L1188

#### Tool used
Manual Review

#### Recommendation
For the first problem use `getLoanLender()` instead of `bid.lender`. Also in `getLoanLender` do not determine the lender by the owner of the NFT, but instead add a mapping that will keep track of who the real lender is.

### [H-02]<a name="h-02"></a> Lender can not close loan with recipient.

A lender can close a loan by calling `lenderCloseLoan` or `lenderCloseLoanWithRecipient`. The first one will withdraw to the lender himself and second one will withdraw to a specified address. However the second will not work as intended and withdraw to the lender again.

#### Vulnerability Detail
Both of the functions call internal function `_lenderCloseLoanWithRecipient`. Then `lenderClaimCollateral` function on the collateral manager is called. However the parameter `_collateralRecipient` is not passed(only 1 parameter can be passed) which means that lender will be the address to withdraw to.

#### Impact
Closing a loan with a recipient will send tokens to the lender.

#### Code Snippet
```solidity
    function _lenderCloseLoanWithRecipient(
        uint256 _bidId,
        address _collateralRecipient
    ) internal acceptedLoan(_bidId, "lenderClaimCollateral") {
        require(isLoanDefaulted(_bidId), "Loan must be defaulted.");

        Bid storage bid = bids[_bidId];
        bid.state = BidState.CLOSED;

        address sender = _msgSenderForMarket(bid.marketplaceId);
        require(sender == bid.lender, "only lender can close loan");

        /*


          address collateralManagerForBid = address(_getCollateralManagerForBid(_bidId)); 

          if( collateralManagerForBid == address(collateralManagerV2) ){
             ICollateralManagerV2(collateralManagerForBid).lenderClaimCollateral(_bidId,_collateralRecipient);
          }else{
             require( _collateralRecipient == address(bid.lender));
             ICollateralManager(collateralManagerForBid).lenderClaimCollateral(_bidId );
          }
          
          */

        collateralManager.lenderClaimCollateral(_bidId);

        emit LoanClosed(_bidId);
    }
```

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L745-L774

#### Tool used
Manual Review

#### Recommendation
Add parameter to `CollateralManager::lenderClaimCollateral` and pass `_collateralRecipient` to it.

### [H-03]<a name="h-03"></a> Borrower can keep loan.

When a borrower repays for his loan, the payment is either send to the lender or to the escrow. The protocol has added `try/catch` block to be sure that if payment can not be send to the lender directly, it will be send to the escrow. However if tokens that return boolean on transfer is used, no one will receive payment and loan will be marked as paid.

#### Vulnerability Detail
The problem is that inside the `try` transferFrom is used instead of safeTransferFrom. Consider the following scenario:

1. Malicious borrower wants a loan for a token that returns boolean on transfer.
2. Lender accepts the bid and lends the borrower tokens.
3. Borrower transfers all tokens to another account and calls `repayLoanFull`.
4. The transferFrom returns false instead of reverting and payment is not send to escrow.

#### Impact
Malicious borrower can get tokens for free.

#### Code Snippet
```solidity
        try 

            bid.loanDetails.lendingToken.transferFrom{ gas: 100000 }(
                _msgSenderForMarket(bid.marketplaceId),
                lender,
                _paymentAmount
            )
        {} catch {
            address sender = _msgSenderForMarket(bid.marketplaceId);

            uint256 balanceBefore = bid.loanDetails.lendingToken.balanceOf(
                address(this)
            ); 

            //if unable, pay to escrow
            bid.loanDetails.lendingToken.safeTransferFrom(
                sender,
                address(this),
                _paymentAmount
            );

            uint256 balanceAfter = bid.loanDetails.lendingToken.balanceOf(
                address(this)
            );

            //used for fee-on-send tokens
            uint256 paymentAmountReceived = balanceAfter - balanceBefore;

            bid.loanDetails.lendingToken.approve(
                address(escrowVault),
                paymentAmountReceived
            );

            IEscrowVault(escrowVault).deposit(
                lender,
                address(bid.loanDetails.lendingToken),
                paymentAmountReceived
            );
        }
```

https://github.com/sherlock-audit/2024-04-teller-finance/blob/main/teller-protocol-v2-audit-2024/packages/contracts/contracts/TellerV2.sol#L909-L948

#### Tool used
Manual Review

#### Recommendation
Use `safeTransferFrom` so it will always revert.