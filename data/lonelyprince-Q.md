### Code: 
https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L199-L226

### Description
In the function `transferToUnoccupiedPlot` only the `latestTaxRate` of the `toilerState` is updated. Since there is a move from one plotId to another, the plotId and lastToilDate should also be updated to newPlotId and new timestamp. If not updated then it will have an impace on the toiler state which is later used to calculate bonus.

```
toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
            .currentTaxRate;
```

### Mitigation
```
toilerState[tokenId].plotId = plotId
toilerState[tokenId].lastToilDate = block.timestamp
```