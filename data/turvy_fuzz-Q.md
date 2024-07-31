## L-1 Inconsistency in the plotId constraint leading to issues with business logic
**Description**
Uses >= when checking during staking:
```solidity
if (plotId >= totalPlotsAvail) revert PlotTooHighError();
```
but uses just < in _farmPlot():
```solidity
if (_getNumPlots(landlord) < _toiler.plotId) {
```
**Recommendation**
Ensure invariants on allowed plotId are consistent

## L-2 The check if (munchablesStaked[mainAccount].length > 10) allows for 11 not 10

**Description**
when staking it checks if (munchablesStaked[mainAccount].length > 10) but before pushing to the array, this allows for 11 and not 10

**Recommendation**
If intended invariant was for max of 10 munchablesStaked then update code to:
```solidity
if (munchablesStaked[mainAccount].length >= 10)
```