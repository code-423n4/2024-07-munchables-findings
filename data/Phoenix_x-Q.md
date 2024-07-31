## Summary
Possibility of assigning negative value to uint variable

## Vulnerability Details
Inside the method `_farmPlots` of the LandManager contract the total schnibbles are calculated using the following formula: `schnibblesTotal = (timestamp - _toiler.lastToilDate) * BASE_SCHNIBBLE_RATE;`
The type of `schnibblesTotal` is uint but the calculation `timestamp - _toiler.lastToilDate` can have a negative value which would assigning negative number to uint type. This is possible only if the variable `timestamp` was assigned the value `plotMetadata[landlord].lastUpdated` and `_toiler.lastToilDate` contains timestamp which is larger than the `timestamp`variable. This is possible only if plotMetadata[landlord].lastUpdated holds an old data and no methods were called to update it with a more recent date.

## Relevant GitHub links:
https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L281

## Impact
Assigning negave value to uint variables which causes the `_farmPlots` method to revert until `plotMetadata[landlord].lastUpdated` gets updated with recent data using methods such as `triggerPlotMetadata` and `updatePlotMetadata`.

## Tools Used
Manual Review

## Recommendations
Change the formula for total schnibbles: `schnibblesTotal = (timestamp - _toiler.lastToilDate) * BASE_SCHNIBBLE_RATE;` in a way that does not rely on `timestamp` having always larger value than _toiler.lastToilDate.
