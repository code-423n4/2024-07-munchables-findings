## [L-01] `landlordMetadata.lastPetMunchable` recording of last timestamp will revert in 2107

[LandManager.sol#L299](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L299)
Updating the last timestamp of a landlord's `lastPetMunchable` will fail to work in 2107 because at that time the value will be bigger than the max a uint32 type can hold.

```solidity
@> landlordMetadata.lastPetMunchable = uint32(timestamp);
```

Using this updated `withdraw()` implementation below renders a fix to this potential issue:

Using a bigger uint type for the `lastPetMunchable` field and then casting it to a bigger uint like below will delay this a lot further in the future

```diff
- landlordMetadata.lastPetMunchable = uint32(timestamp);
+ landlordMetadata.lastPetMunchable = uint64(timestamp);
```

## [L-02] Transferring tokens across different landlords is not possible and only constrained to same landlord plots

[LandManager.sol#L199-L226](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226)

```solidity
function transferToUnoccupiedPlot(
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        ToilerState memory _toiler = toilerState[tokenId];
        uint256 oldPlotId = _toiler.plotId;
 @>       uint256 totalPlotsAvail = _getNumPlots(_toiler.landlord);
        if (_toiler.landlord == address(0)) revert NotStakedError();
        if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();
 @>       if (plotOccupied[_toiler.landlord][plotId].occupied)
 @>           revert OccupiedPlotError(_toiler.landlord, plotId);
 @>       if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
            .currentTaxRate;
        plotOccupied[_toiler.landlord][oldPlotId] = Plot({
            occupied: false,
            tokenId: 0
        });
@>       plotOccupied[_toiler.landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        emit FarmPlotLeave(_toiler.landlord, tokenId, oldPlotId);
        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }
```

While this is not so much of a big deal in itself, it is better to allow transfers to any other landlord's plot as long as it is unoccupied and not constrain transfers between same landlord. Users do get a workaround by first unstaking and then restaking to the supposed new landlord.

## [L-03] Staking on the last plot is not possible

[LandManager.sol#L146](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L146)

While the current implementation is trying to mitigate against landlords having 0 weights (resulting from not locking tokens) and renters being able stake on `plotId` 0 thus both (renter and landlord) earning schnibbles, it also introduces an edge case/off by one error whereby if a landlord has 10 weights, staking on the `plotId` 10 will be impossible and thus only plots 0-9 are available.

```solidity
function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        ...

        uint256 totalPlotsAvail = _getNumPlots(landlord);
@>        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        ...
    }
```

Instead of having to give up each landlords last plot, a better way to handle this is to check that each landlord a renter is staking on has some weights. That is, when we make the call to `_getNumPlots(landlord)`, we check that the `totalPlotsAvail` returned is non zero. If it is 0, then it means the landlord's locked tokens e.g USDB, WETH is not sufficient enough to start earning tax after being divvy up by `PRICE_PER_PLOT`. That would then allow all of a landlord's plot to earn tax by getting rid of the `=` in the check here `if (plotId >= totalPlotsAvail)`

```diff
uint256 totalPlotsAvail = _getNumPlots(landlord);
+ if (totalPlotsAvail == 0) revert();
- if (plotId >= totalPlotsAvail) revert PlotTooHighError();
+ if (plotId > totalPlotsAvail) revert PlotTooHighError();
```