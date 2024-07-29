## Title: Inability to Unstake Munchables When Contract is Paused

## **Relevant GitHub Links:**

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L173

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L40

## **Vulnerability Details:**

The [unstakeMunchable](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L173) function allows users to unstake their Munchable from a plot. It first calculates the user's latest rewards using the [forceFarmPlots](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L40) modifier and then sends the NFT back to the user. This function has the notPaused modifier, meaning if the contract is paused for any reason, users will not be able to unstake from the contract.

When the contract is paused, users are unable to perform the unstake operation, which can lead to several issues, including being locked into a specific tax rate or being unable to list their tokens for sale or other purposes.

```solidity
    function unstakeMunchable(uint256 tokenId) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount,) = _getMainAccountRequireRegistered(msg.sender);
        ToilerState memory _toiler = toilerState[tokenId];
        if (_toiler.landlord == address(0)) revert NotStakedError();
        if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();

        plotOccupied[_toiler.landlord][_toiler.plotId] = Plot({occupied: false, tokenId: 0});
        toilerState[tokenId] =
            ToilerState({lastToilDate: 0, plotId: 0, landlord: address(0), latestTaxRate: 0, dirty: false});
        munchableOwner[tokenId] = address(0);
        _removeTokenIdFromStakedList(mainAccount, tokenId);

        munchNFT.transferFrom(address(this), mainAccount, tokenId);
        emit FarmPlotLeave(_toiler.landlord, tokenId, _toiler.plotId);
    }
```

## **Impact:**

If the contract is paused, users will be unable to unstake their tokens, resulting in the following issues:

- Users will be locked into a specific tax rate.
- If the landlord unlocks funds, users will be stuck in an invalid plot, receiving no rewards.
- Users will not be able to unstake their tokens to sell them on a marketplace.

## **Tools Used:**

Manual analysis

## **Recommendation:**

Consider implementing a more granular pause mechanism that allows specific functions to be paused independently. For example, using a mapping to control the paused state of individual functions can enable the contract owner to pause certain operations while still allowing users to unstake their tokens.