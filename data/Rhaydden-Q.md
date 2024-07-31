
# QA for Munchables

## Table of Contents

| Issue ID | Description |
| -------- | ----------- |
| [QA-01](#qa-01-lingering-token-approvals-in-landmanager-may-lead-to-unauthorized-staking-of-munchables) | Lingering token approvals in `LandManager` may lead to unauthorized staking of Munchables |
| [QA-02](#qa-02-transfertounoccupiedplot-fails-to-update-toilerstateplotid-leading-to-incorrect-plot-tracking-and-could-also-unnecessarily-mark-plots-as-dirty) | `transferToUnoccupiedPlot` fails to update `toilerState.plotId`, leading to incorrect plot tracking and could also unnecessarily mark plots as "dirty" |
| [QA-03](#qa-03-tax-rate-updates-bypass-timestamp-refresh-distorting-schnibble-calculations-for-invalid-plots) | Tax rate updates bypass `Timestamp` refresh, distorting Schnibble calculations for invalid plots |
| [QA-04](#qa-04-improper-initialization-in-landmanager-allows-partial-state-setup-risking-inconsistent-behavior) | Improper initialization in `LandManager` allows partial state setup, risking inconsistent behavior |
| [QA-05](#qa-05-inaccurate-timestamp-handling-in-_farmplots-function-leading-to-potential-inaccurate-schnibbles-calculation) | Inaccurate timestamp handling in `_farmPlots` function leading to potential inaccurate Schnibbles calculation |
| [QA-06](#qa-06-repurposed-storagekeys-in-landmanager-compromise-code-clarity) | Repurposed StorageKeys in `LandManager` compromise code clarity |
| [QA-07](#qa-07-incorrect-staking-limit-check-allows-users-to-stake-11-munchables-instead-of-intended-10) | Incorrect staking limit check allows users to stake `11` munchables instead of intended `10` |
| [QA-08](#qa-08-precision-loss-in-landlord-schnibble-calculation) | Precision loss in landlord Schnibble calculation |

## [QA-01] Lingering token approvals in `LandManager` may lead to unauthorized staking of Munchables

#### Impact
The `LandManager` contract lacks a mechanism to revoke or reset token approvals after they are granted. This could potentially allow the contract to stake Munchables without the current intent of the token owner, if they forget to revoke approvals directly through the `MunchNFT` contract. 

>The impact is low, as it doesn't result in direct fund loss but could lead to unexpected staking of valuable NFTs.

#### Proof of Concept
In the `stakeMunchable` function of the LandManager contract, there's a check for token approval:
https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L148-L151

```solidity
if (
    !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
    munchNFT.getApproved(tokenId) != address(this)
) revert NotApprovedError();
```

This check allows the staking operation to proceed if either a blanket approval for all tokens or a specific token approval is in place. However, there are two key issues:

1. The contract doesn't provide a way to revoke these approvals within its own functions.
2. There's no mechanism to reset or check for revoked approvals before subsequent staking operations.

For example, if a user grants approval and then later wants to revoke it, they must do so through the `MunchNFT` contract directly. If they forget to do this, the `LandManager` contract will still consider itself approved for future staking operations.

#### Recommended Mitigation Steps
Consider implementing a function in the `LandManager` that allows users to revoke approvals for their tokens, which would interact with the `MunchNFT` contract to remove the approvals. Also, before each staking operation, add a fresh approval check by calling the `MunchNFT` contract directly, rather than relying on potentially outdated approval states. Or consider implementing an approval expiration mechanism, where approvals granted to the LandManager contract automatically expire after a certain period or after specific conditions are met (e.g., after unstaking).


## [QA-02] `transferToUnoccupiedPlot` fails to update `toilerState.plotId`, leading to incorrect plot tracking and could also unnecessarily mark plots as "dirty"

#### Impact

The `transferToUnoccupiedPlot` function in the LandManager contract fails to update the `plotId` in the `toilerState` mapping when a Munchable NFT is moved to a new plot resulting in a discrepancy between the actual plot a token is associated with and the plot recorded in the `toilerState`. 

#### Proof of Concept

Look at the `transferToUnoccupiedPlot` function: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L199-L227

```solidity
function transferToUnoccupiedPlot(
    uint256 tokenId,
    uint256 plotId
) external override forceFarmPlots(msg.sender) notPaused {
    (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
    ToilerState memory _toiler = toilerState[tokenId];
    uint256 oldPlotId = _toiler.plotId;
    uint256 totalPlotsAvail = _getNumPlots(_toiler.landlord);
    // ... (validity checks)

    toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
        .currentTaxRate;
    plotOccupied[_toiler.landlord][oldPlotId] = Plot({
        occupied: false,
        tokenId: 0
    });
    plotOccupied[_toiler.landlord][plotId] = Plot({
        occupied: true,
        tokenId: tokenId
    });

    emit FarmPlotLeave(_toiler.landlord, tokenId, oldPlotId);
    emit FarmPlotTaken(toilerState[tokenId], tokenId);
}
```

The function updates the `plotOccupied` mapping for both the old and new plots, correctly marking the old plot as unoccupied and the new plot as occupied. But, it fails to update the `plotId` in the `toilerState` mapping for the token.

This means that while `plotOccupied` correctly reflects the new state, `toilerState[tokenId].plotId` still contains the old plot ID. This could lead to issues in other parts of the protocol that rely on the `toilerState` mapping, such as the [`_farmPlots` function](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L250):

```solidity
function _farmPlots(address _sender) internal {
    // ...
    for (uint8 i = 0; i < staked.length; i++) {
        // ...
        _toiler = toilerState[tokenId];
  @>      // ...
        if (_getNumPlots(landlord) < _toiler.plotId) {
            timestamp = plotMetadata[landlord].lastUpdated;
            toilerState[tokenId].dirty = true;
        }
        // ... (farming calculations using _toiler.plotId)
    }
    // ...
}
```

In this function, the outdated `plotId` in `toilerState` could lead to incorrect farming calculations or unnecessary marking of plots as "dirty".

#### Recommended Mitigation Steps

Update the `transferToUnoccupiedPlot` function to include an update to the `toilerState` mapping. Add the following line after updating the `plotOccupied` mappings and before emitting the events:

```solidity
toilerState[tokenId].plotId = plotId;
```

This will make sure that the `toilerState` mapping is correctly updated with the new plot ID when a token is transferred to an unoccupied plot. Additionally, consider adding a similar update in any other functions that might move a Munchable to a different plot. 









## [QA-03] Tax rate updates bypass `Timestamp` refresh, distorting Schnibble calculations for invalid plots

#### Impact
`updateTaxRate` function  fails to update the `lastUpdated` timestamp when changing a landlord's tax rate. This can lead to incorrect Schnibble calculations in the `_farmPlots` function. The impact is particularly significant when dealing with plots that are no longer valid, as the function relies on the outdated `lastUpdated` timestamp for calculations.

#### Proof of Concept
Take a look at `updateTaxRate` function: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L92-L101

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

This function updates the `currentTaxRate` but doesn't update the `lastUpdated` timestamp in `plotMetadata`.

The problem manifests in the `_farmPlots` function: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L258-L260

```solidity
if (_getNumPlots(landlord) < _toiler.plotId) {
    timestamp = plotMetadata[landlord].lastUpdated;
    toilerState[tokenId].dirty = true;
}
```

Here, for plots that are no longer valid, the function uses `plotMetadata[landlord].lastUpdated` as the timestamp. If tax rates have changed since the last update which is possible to `lastUpdated`, this could lead to incorrect Schnibble calculations.



#### Recommended Mitigation Steps
Update the `updateTaxRate` function to include the `lastUpdated` timestamp update:

```diff
function updateTaxRate(uint256 newTaxRate) external override notPaused {
    (address landlord, ) = _getMainAccountRequireRegistered(msg.sender);
    if (newTaxRate < MIN_TAX_RATE || newTaxRate > MAX_TAX_RATE)
        revert InvalidTaxRateError();
    if (plotMetadata[landlord].lastUpdated == 0)
        revert PlotMetadataNotUpdatedError();
    uint256 oldTaxRate = plotMetadata[landlord].currentTaxRate;
    plotMetadata[landlord].currentTaxRate = newTaxRate;
+   plotMetadata[landlord].lastUpdated = block.timestamp;
    emit TaxRateChanged(landlord, oldTaxRate, newTaxRate);
}
```








## [QA-04] Improper initialization in `LandManager` allows partial state setup, risking inconsistent behavior

#### Impact

The current initialization pattern in the `LandManager` contract and its inherited contracts _(although out of scope for this audit)_ creates a issue where partial initialization of the contract state is possible. This can lead to an inconsistent contract state, potentially causing unexpected behavior.

#### Proof of Concept

The issue stems from the inheritance structure and initialization pattern used in the `LandManager` contract:

[LandManager](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L11) inherits from `BaseBlastManagerUpgradeable`: 

```solidity
contract LandManager is BaseBlastManagerUpgradeable, ILandManager {
    // ...
}
```

`BaseBlastManagerUpgradeable` in turn [inherits](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/BaseBlastManagerUpgradeable.sol#L13-L15) from `BaseConfigStorageUpgradeable`:

```solidity
abstract contract BaseBlastManagerUpgradeable is
    BaseBlastManager,
    BaseConfigStorageUpgradeable
{
    // ...
}
```

But both parent contracts have their own `initialize` functions with the `initializer` modifier:

```solidity
// In BaseConfigStorageUpgradeable
function initialize(address _configStorage) public virtual initializer {
    __BaseConfigStorage_setConfigStorage(_configStorage);
}

// In BaseBlastManagerUpgradeable
function initialize(address _configStorage) public virtual override initializer {
    BaseConfigStorageUpgradeable.initialize(_configStorage);
}
```

However, the `LandManager` contract overrides the [`initialize` function:](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L45-L48)

```solidity
function initialize(address _configStorage) public override initializer {
    BaseBlastManagerUpgradeable.initialize(_configStorage);
    _reconfigure();
}
```

Now, the issue here is that while the `initializer` modifier in `LandManager` prevents multiple calls to its `initialize` function, it doesn't prevent direct calls to the `initialize` functions of parent contracts. This could lead to partial initialization if, for example, `BaseConfigStorageUpgradeable.initialize` is called directly, bypassing the `initializer` check in LandManager.

#### Recommended Mitigation Steps

We could consider using the `__init` pattern for internal initializers in parent contracts. For example, we could rename `initialize` to `__BaseConfigStorageUpgradeable_init` in BaseConfigStorageUpgradeable and make it `internal`.
In the LandManager's `initialize` function, call all necessary internal initializers from parent contracts explicitly. And ensure that the `initializer` modifier is only used in the most derived contract (LandManager in this case).








## [QA-05] Inaccurate timestamp handling in `_farmPlots` function leading to potential inaccurate Schnibbles calculation

#### Proof of Concept

>First of all, I'd like to note that this issue is considered out of scope BUT the sponsors said they will accept alternative solutions to mitigate this issue as it is clearly stated in the "Known issues" in the [ReadMe](https://github.com/code-423n4/2024-07-munchables#automated-findings--publicly-known-issues):


"Within the _farmPlots() function, the following edge case can happen: If the plot ID is lower than the max plot amount, the system uses the last updated plot metadata time and tracks this with a dirty boolean flag. The edge case occurs when a user hasn’t farmed in a while and the landlord updates the plots multiple times (ie unlocks/locks multiple times), causing the last updated time to reflect the latest lock update rather than the first. This is a known edge case that we are comfortable with. HOWEVER, we will accept alternative solutions to mitigate this issue. Pointing it out will not result in a valid finding."


The `_farmPlots` function in the `LandManager` contract uses the `lastUpdated` timestamp from `plotMetadata` to calculate the schnibbles earned by a renter. However, if the plot ID is greater than the number of plots available to the landlord, the function uses the latest `lastUpdated` timestamp, which can lead to inaccurate calculations if the landlord has updated the plots multiple times.

Look here: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L232-L311

```solidity
function _farmPlots(address _sender) internal {
    (
        address mainAccount,
        MunchablesCommonLib.Player memory renterMetadata
    ) = _getMainAccountRequireRegistered(_sender);

    uint256[] memory staked = munchablesStaked[mainAccount];
    MunchablesCommonLib.NFTImmutableAttributes memory immutableAttributes;
    ToilerState memory _toiler;
    uint256 timestamp;
    address landlord;
    uint256 tokenId;
    int256 finalBonus;
    uint256 schnibblesTotal;
    uint256 schnibblesLandlord;
    for (uint8 i = 0; i < staked.length; i++) {
        timestamp = block.timestamp;
        tokenId = staked[i];
        _toiler = toilerState[tokenId];
        if (_toiler.dirty) continue;
        landlord = _toiler.landlord;
        // use last updated plot metadata time if the plot id doesn't fit
        // track a dirty bool to signify this was done once
        // the edge case where this doesnt work is if the user hasnt farmed in a while and the landlord
        // updates their plots multiple times. then the last updated time will be the last time they updated their plot details
        // instead of the first
        if (_getNumPlots(landlord) < _toiler.plotId) {
            timestamp = plotMetadata[landlord].lastUpdated;
            toilerState[tokenId].dirty = true;
        }
        // existing code...
    }
    // existing code...
}
```

#### Impact

The impact of this issue is that the `schnibblesTotal` calculation may be based on an incorrect `timestamp`, leading to either overestimating or underestimating the schnibbles earned by the renter and the landlord. This can result in unfair distribution of rewards and potential dissatisfaction among users.

#### Recommended Mitigation Steps

Here are 3 steps that could be taken to fix this:

##### Solution 1: Track Multiple Timestamps

1. **Modify `PlotMetadata` to include an array of timestamps:**

```solidity
struct PlotMetadata {
    uint256[] updateTimestamps;
    uint256 currentTaxRate;
}
```

2. **Update `_reconfigure` and other functions to handle multiple timestamps:**

```solidity
function _reconfigure() internal {
    // existing code...
    plotMetadata[landlord].updateTimestamps.push(block.timestamp);
    // existing code...
}
```

3. **Modify `_farmPlots` to use the correct timestamp:**

```solidity
function _farmPlots(address _sender) internal {
    // existing code...
    for (uint8 i = 0; i < staked.length; i++) {
        // existing code...
        if (_getNumPlots(landlord) < _toiler.plotId) {
            uint256 correctTimestamp = _getCorrectTimestamp(landlord, _toiler.lastToilDate);
            timestamp = correctTimestamp;
            toilerState[tokenId].dirty = true;
        }
        // existing code...
    }
    // existing code...
}

function _getCorrectTimestamp(address landlord, uint256 lastToilDate) internal view returns (uint256) {
    uint256[] memory timestamps = plotMetadata[landlord].updateTimestamps;
    for (uint256 i = 0; i < timestamps.length; i++) {
        if (timestamps[i] > lastToilDate) {
            return timestamps[i];
        }
    }
    return block.timestamp; // fallback to current timestamp if no match found
}
```

##### Solution 2: Use a Mapping for Timestamps

1. **Modify `PlotMetadata` to include a mapping of timestamps:**

```solidity
struct PlotMetadata {
    mapping(uint256 => uint256) updateTimestamps;
    uint256 currentTaxRate;
}
```

2. **Update `_reconfigure` and other functions to handle the mapping:**

```solidity
function _reconfigure() internal {
    // existing code...
    plotMetadata[landlord].updateTimestamps[plotId] = block.timestamp;
    // existing code...
}
```

3. **Modify `_farmPlots` to use the correct timestamp:**

```solidity
function _farmPlots(address _sender) internal {
    // existing code...
    for (uint8 i = 0; i < staked.length; i++) {
        // existing code...
        if (_getNumPlots(landlord) < _toiler.plotId) {
            uint256 correctTimestamp = plotMetadata[landlord].updateTimestamps[_toiler.plotId];
            timestamp = correctTimestamp;
            toilerState[tokenId].dirty = true;
        }
        // existing code...
    }
    // existing code...
}
```

##### Solution 3: Snapshot Mechanism

1. **Create a `Snapshot` struct to store plot states:**

```solidity
struct Snapshot {
    uint256 timestamp;
    uint256 taxRate;
}
```

2. **Modify `PlotMetadata` to include an array of snapshots:**

```solidity
struct PlotMetadata {
    Snapshot[] snapshots;
}
```

3. **Update `_reconfigure` and other functions to handle snapshots:**

```solidity
function _reconfigure() internal {
    // existing code...
    plotMetadata[landlord].snapshots.push(Snapshot({
        timestamp: block.timestamp,
        taxRate: currentTaxRate
    }));
    // existing code...
}
```

4. **Modify `_farmPlots` to use the correct snapshot:**

```solidity
function _farmPlots(address _sender) internal {
    // existing code...
    for (uint8 i = 0; i < staked.length; i++) {
        // existing code...
        if (_getNumPlots(landlord) < _toiler.plotId) {
            Snapshot memory correctSnapshot = _getCorrectSnapshot(landlord, _toiler.lastToilDate);
            timestamp = correctSnapshot.timestamp;
            toilerState[tokenId].dirty = true;
        }
        // existing code...
    }
    // existing code...
}

function _getCorrectSnapshot(address landlord, uint256 lastToilDate) internal view returns (Snapshot memory) {
    Snapshot[] memory snapshots = plotMetadata[landlord].snapshots;
    for (uint256 i = 0; i < snapshots.length; i++) {
        if (snapshots[i].timestamp > lastToilDate) {
            return snapshots[i];
        }
    }
    return Snapshot({timestamp: block.timestamp, taxRate: 0}); // fallback to current state if no match found
}
```

Any of the recommendations above should fix this issue












## [QA-06] Repurposed StorageKeys in `LandManager` compromise code clarity

#### Impact
>I threw a question to the sponsor in the discord PT and their response was:
>
>"This is because the LandManager is being launched after the rest of the contracts. We are unable to update the StorageKeys so we have to use ones that are unused for the data types we need."

However, the use of unrelated `StorageKey` values for tax rate configurations in the `LandManager`  reduces code readability, increases the risk of future maintenance errors, and could potentially lead to conflicts if these keys are needed for their original purposes in future updates. Although not a bug per se, this approach introduces subtle risks to the long-term maintainability and extensibility of the system.

#### Proof of Concept
In the LandManager contract, tax rates are configured using seemingly unrelated StorageKey values: 
https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L65-L72

```solidity
// LandManager.sol
function _reconfigure() internal {
    // ...
    MIN_TAX_RATE = IConfigStorage(configStorage).getUint(
        StorageKey.LockManager
    );
    MAX_TAX_RATE = IConfigStorage(configStorage).getUint(
        StorageKey.AccountManager
    );
    DEFAULT_TAX_RATE = IConfigStorage(configStorage).getUint(
        StorageKey.ClaimManager
    );
    // ...
}
```

However, in the IConfigStorage interface, these StorageKey values are defined for different purposes:

```solidity
// IConfigStorage.sol
enum StorageKey {
    // ...
    LockManager,
    AccountManager,
    ClaimManager,
    // ...
}
```


While this approach solves the immediate problem of storing new configuration values without updating the StorageKey enum, it introduces subtle issues:

- The StorageKey names do not reflect their actual usage in LandManager, making the code less intuitive.
- Future developers might misinterpret the purpose of these keys, leading to potential errors.
- If these StorageKey values are needed for their original purposes in future updates, it could cause conflicts.

#### Recommended Mitigation Steps
Consider adding  detailed comments in both `LandManager` and `IConfigStorage` contracts explaining the repurposing of these StorageKey values for LandManager's tax rates. Also implement additional checks or assertions in the configuration retrieval process to ensure these StorageKey values are used correctly in different contexts.



## [QA-07] Incorrect staking limit check allows users to stake `11` munchables instead of intended `10`

#### Impact
The current implementation allows users to stake up to 11 Munchables, exceeding the intended maximum of 10. This could lead to an imbalance in the game protocol, potentially giving some users an unfair advantage by allowing them to earn more rewards than intended. 

#### Proof of Concept
https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L140

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        // ... other checks ...
 @>     if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        // ... rest of the function
}
```

The condition `munchablesStaked[mainAccount].length > 10` only reverts when trying to stake the 12th Munchable. This means:

1. A user can successfully stake their 1st to 10th Munchable.
2. They can also stake their 11th Munchable, as 11 is not greater than 10.
3. The error is only thrown when trying to stake the 12th Munchable.

As a result, users can stake 11 Munchables instead of the intended maximum of 10.

#### Recommended Mitigation Steps
Change the condition to use greater than or equal to (`>=`) instead of strictly greater than (`>`):

```diff
function stakeMunchable(
    address landlord,
    uint256 tokenId,
    uint256 plotId
) external override forceFarmPlots(msg.sender) notPaused {
    // ... other checks ...
-   if (munchablesStaked[mainAccount].length > 10)
+   if (munchablesStaked[mainAccount].length >= 10)
        revert TooManyStakedMunchiesError();
    // ... rest of the function
}
```


## [QA-08] Precision loss in landlord Schnibble calculation

#### Impact
The current implementation of the schnibble calculation for landlords can result in significant precision loss, potentially leading to landlords receiving fewer schnibbles than they should. In extreme cases, particularly with small `schnibblesTotal` values or low tax rates, landlords might receive no schnibbles at all when they should have received a small amount.

#### Proof of Concept

https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L279-L289

```solidity
    schnibblesTotal =
        (timestamp - _toiler.lastToilDate) *
        BASE_SCHNIBBLE_RATE;
    schnibblesTotal = uint256(
        (int256(schnibblesTotal) +
            (int256(schnibblesTotal) * finalBonus)) / 100
    );
 ❌  schnibblesLandlord =
        (schnibblesTotal * _toiler.latestTaxRate) /
        1e18;
```

The issue lies in how the calculation of `schnibblesLandlord` is done. The `_toiler.latestTaxRate` is likely represented as a value between `0` and `1e18` (where 1e18 is 100%). When multiplying `schnibblesTotal` by `_toiler.latestTaxRate`, the result is a very large number due to the tax rate's 18 decimal places. The subsequent division by 1e18 can lead to significant loss of precision or even round down to zero for small values of `schnibblesTotal` or low tax rates.

Forr example:
Let's say `schnibblesTotal` is 100 and the tax rate is 1% (represented as 1e16 in `LandManager`).

Current calculation will be:
```
schnibblesLandlord = (100 * 1e16) / 1e18 = 1000000000000000000 / 1e18 = 1
```

Expected calculation should be:
```
schnibblesLandlord = 100 * (1e16 / 1e18) = 100 * 0.01 = 1
```

Although in this case, the result is correct. Albeit, if `schnibblesTotal` were 99 instead:

Current calculation:
```
schnibblesLandlord = (99 * 1e16) / 1e18 = 990000000000000000 / 1e18 = 0
```

Expected calculation:
```
schnibblesLandlord = 99 * (1e16 / 1e18) = 99 * 0.01 = 0.99
```

In this case, the landlord receives 0 schnibbles instead of the expected 0.99 and will be on the losing end.

#### Recommended Mitigation Steps
Rearrange the calculation to minimize precision loss: 
```diff
- schnibblesLandlord =
-     (schnibblesTotal * _toiler.latestTaxRate) /
-     1e18;
+ schnibblesLandlord = schnibblesTotal * (_toiler.latestTaxRate / 1e18);
```

For even greater precision, consider implementing a fixed-point math library specifically designed for these types of calculations.


