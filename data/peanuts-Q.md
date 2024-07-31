## Summary of Contents

| Index | Title                                                                                                           |
| ----- | --------------------------------------------------------------------------------------------------------------- |
| L-01  | Inconsistent casting from uint8 to int8 when calculating RARITY_BONUS                                           |
| L-02  | Insufficient validation of `finalBonus` in `_farmPlots()`                                                       |
| L-03  | Updating currentTaxRate in `transferToUnoccupiedPlot()` in redundant as it is already updated in `_farmPlots()` |
| L-04  | plotId can be frontrunned                                                                                       |
| L-05  | The tax rate is unfair for existing users if lowered after an arbitrary time                                    |
| L-06  | plotId and tokenId can be zero                                                                                  |

### [L-01] Inconsistent casting from uint8 to int8 when calculating RARITY_BONUS

`RARITY_BONUS` is defined as such:

```
    uint8[] RARITY_BONUSES;
```

When calculating, `RARITY_BONUS` is casted to an `int8`, which is inconsistent from the original definition

```
  int16(int8(RARITY_BONUSES[uint256(immutableAttributes.rarity)]));
```

Ensure that the casting is not inconsistent. Since `RARITY_BONUS` is always assumed to be positive, it is ok to leave it as a uint.

Code: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L278

### [L-02] Insufficient validation of `finalBonus` in `_farmPlots()`

It is unsure what the min and max of the `finalBonus` should be, as it is set as an `int256` but later downcasted to a `int16` when calculating the `REALM_BONUS` and `RARITY_BONUS`.

In the calculation, `schnibblesTotal` is casted to `uint256`, which assumes that the positivity or negativity of `finalBonus` should not matter.

```
 schnibblesTotal = uint256((int256(schnibblesTotal) + (int256(schnibblesTotal) * finalBonus)) / 100);
```

`finalBonus` also does not have a upper bound validation check, so the bonus can be over 100%. If not intended, the function should have a check to ensure that `finalBonus` is not > 100, or if it is greater, set it to 100% max.

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L283-L285

### [L-03] Updating currentTaxRate in `transferToUnoccupiedPlot()` in redundant as it is already updated in `_farmPlots()`

In `transferToUnoccupiedPlot()`, `toilerState[tokenId].latestTaxRate` is updated to the latest `currentTaxRate` of the `landlord`.

```
toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord].currentTaxRate;
```

However, in `_farmPlots()`, which is called by the modifier `forceFarmPlots()`, updates the `latestTaxRate` as well.

```
 toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord].currentTaxRate;
```

Consider removing the line in `transferToUnoccupiedPlot()`.

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L213-L214

### [L-04] plotId can be frontrunned

Anybody can stake into any `plotId` if it is not occupied. If `plotId` matters in the game (maybe a certain number eg 1337 is an ultra rare plot), then users can frontrun others and stake into the plotId. Also, if all `plotIds` are the same, users can simply stake on an extremely low `plotId` number since the higher it is, the more chance for it to be dirty.

```
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
>       uint256 plotId @audit -> anybody can use any number
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
```

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L142

### [L-05] The tax rate is unfair for existing users if lowered after an arbitrary time

The landlord can change the tax rate through `updateTaxRate`. If the landlord starts with a high tax rate and reduces it midway, users still have to pay the high tax rates.

At time T1, tax rates is set at 20% for landlord A. User Alice stakes into the landlord's plot.

At time T2, landlord A reduces tax rates to 5%. Alice does not know about this reduction.

A few weeks later, at time T3, Alice decides to unstake from the plot. She has to pay 20% tax from time T1 to T2, and also from time T2 to T3.

Bob stakes into the landlord's plot at time T2. At time T3, he unstakes. He only pays 5% tax from time T2 to T3.

```
   schnibblesLandlord =
                (schnibblesTotal * _toiler.latestTaxRate) /
                1e18;

            toilerState[tokenId].lastToilDate = timestamp;
            toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
                .currentTaxRate;

            renterMetadata.unfedSchnibbles += (schnibblesTotal -
                schnibblesLandlord);
```

### [L-06] plotId and tokenId can be zero

Users can choose to stake into `plotId` 0 and also `tokenId` of an NFT can be zero, which may cause confusion when checking the toilerState / plotOccupied of a particular `tokenId`. When unstaking, `tokenId` and `plotId` is set to zero.

```
        plotOccupied[_toiler.landlord][_toiler.plotId] = Plot({
            occupied: false,
            tokenId: 0
        });
        toilerState[tokenId] = ToilerState({
            lastToilDate: 0,
            plotId: 0,
            landlord: address(0),
            latestTaxRate: 0,
            dirty: false
        });
```

A potential solution is to set to uint256.max to refer to an unstaked `tokenId` or an unoccupied plot.

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L164
