## [L-01] `updateTaxRate` does not update `plotMetadata[landlord].lastUpdated`

The purpose of `updateTaxRate` is to update the tax rate of the `plotMetadata` for the caller.

The `plotMetadata` struct includes two fields: `lastUpdated` and `currentTaxRate`. The `currentTaxRate` represents the tax rate currently set for the caller’s plots, while `lastUpdated` indicates the last time the plot metadata was updated.

```solidity
    struct PlotMetadata {
        uint256 lastUpdated;
        uint256 currentTaxRate;
    }
```

However, the current implementation of `updateTaxRate` updates only the `currentTaxRate` field, without modifying `lastUpdated`.

```solidity
    function updateTaxRate(uint256 newTaxRate) external override notPaused {
        (address landlord, ) = _getMainAccountRequireRegistered(msg.sender); 
        if (newTaxRate < MIN_TAX_RATE || newTaxRate > MAX_TAX_RATE)
            revert InvalidTaxRateError();
        if (plotMetadata[landlord].lastUpdated == 0)
            revert PlotMetadataNotUpdatedError();
        uint256 oldTaxRate = plotMetadata[landlord].currentTaxRate;
        plotMetadata[landlord].currentTaxRate = newTaxRate;
        emit TaxRateChanged(landlord, oldTaxRate, newTaxRate);
    }
```

### Mitigation: 
To properly reflect that `plotMetadata` has been updated, you should also update the `lastUpdated` field. Add the following line to set `lastUpdated` to the current `block.timestamp`:

```diff
    function updateTaxRate(uint256 newTaxRate) external override notPaused {
        (address landlord, ) = _getMainAccountRequireRegistered(msg.sender); // @audit Is there a case where address landlord, wouldn't match msg.sender?
        if (newTaxRate < MIN_TAX_RATE || newTaxRate > MAX_TAX_RATE)
            revert InvalidTaxRateError();
        if (plotMetadata[landlord].lastUpdated == 0)
            revert PlotMetadataNotUpdatedError();
        uint256 oldTaxRate = plotMetadata[landlord].currentTaxRate;
        plotMetadata[landlord].currentTaxRate = newTaxRate;
+       plotMetadata[landlord].lastUpdated = block.timestamp;
        emit TaxRateChanged(landlord, oldTaxRate, newTaxRate);
    }
```

## [L-02] Anyone could be a landlord, even if they haven't staked any amount

The purpose of the [`LandManager.triggerPlotMetadata()`](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L104-L114) function is to provide `plotMetadata` to accounts that had locked funds before the `LandManager` was deployed. 

```solidity
function triggerPlotMetadata() external override notPaused {
    (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
    if (plotMetadata[mainAccount].lastUpdated != 0)
        revert PlotMetadataTriggeredError();
    plotMetadata[mainAccount] = PlotMetadata({
        lastUpdated: block.timestamp,
        currentTaxRate: DEFAULT_TAX_RATE
    });

    emit UpdatePlotsMeta(mainAccount);
}
```

The issue is that the function does not check if the `msg.sender` has staked any amount, allowing anyone to become a landlord.

### Mitigation: 
Check if the `mainAccount` has any staked funds before giving them `plotMetadata`.