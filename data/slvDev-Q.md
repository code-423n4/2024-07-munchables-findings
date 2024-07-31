## [L-01] Unnecessary state update in `stakeMunchable()` for first-time stakers
The `stakeMunchable()` function calls the `forceFarmPlots` modifier before execution:
```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
```
This modifier executes the internal `_farmPlots(_account)` function, which performs the following steps:
1. Retrieves the user's metadata
2. Gets all staked munchables for the user
3. Iterates through the staked munchables (if any)
4. Updates the user's metadata
For a first-time staker, the list of staked munchables will be empty, so the loop won't execute. 
However, the function still updates the user's metadata at the end, even though no changes have been made:
```solidity
function _farmPlots(address _sender) internal {
        (
            address mainAccount,
            MunchablesCommonLib.Player memory renterMetadata
        ) = _getMainAccountRequireRegistered(_sender);

        uint256[] memory staked = munchablesStaked[mainAccount];
        /// ...
        for (uint8 i = 0; i < staked.length; i++) {
            /// ...
        }
@>      accountManager.updatePlayer(mainAccount, renterMetadata);
    }
```
### Recomendtion
add `if (staked.length == 0) return;` this will prevent user to pay extra money for the first stake.

## [L-02] `updatePlotMetadata()` can be executed when `LandManager` is paused
Even though this function can only be called by the AccountManager, it should also include the `notPaused` modifier to prevent execution when the system is paused.
```solidity
    function updatePlotMetadata(
        address landlord
    ) external override onlyConfiguredContract(StorageKey.AccountManager) {
```

## [L-03] Upgradeable `LandManager` missing `__gap[50]` storage variable
In the context of upgradeable contracts, ensuring forward compatibility between different contract versions is a critical concern. 
One key strategy for ensuring this compatibility is the use of a `__gap` storage variable. 
Without the `__gap` storage variable, adding new state variables in a contract upgrade can risk overwriting the existing contract storage.
```solidity
contract LandManager is BaseBlastManagerUpgradeable, ILandManager
```

## [L-04] It's better to use `safeTransferFrom()` when send ERC721 to user
If the mainAccount is a contract, `transferFrom()` might lock the NFT in that account.
It's better to use `safeTransferFrom()` with a non-reentrancy modifier.

```solidity
    function unstakeMunchable(
        uint256 tokenId
    ) external override forceFarmPlots(msg.sender) notPaused {
        // ..
        munchNFT.transferFrom(address(this), mainAccount, tokenId);
        //..
    }
```

## [L-04] unsafe convertion from `uint256` to `int256`
If the product `(timestamp - _toiler.lastToilDate) * BASE_SCHNIBBLE_RATE` exceeds the maximum value for `int256`, next calculation will be incorrect due to casting overflow.

```solidity
    uint256 schnibblesTotal;
    // ...
    schnibblesTotal = (timestamp - _toiler.lastToilDate) * BASE_SCHNIBBLE_RATE;
    schnibblesTotal = uint256(
        (int256(schnibblesTotal) + (int256(schnibblesTotal) * finalBonus)) / 100
    );
```

## [N-01] No need to set each field to zero in `unstakeMunchable()`
`delete` is more gas efficient and makes code more readable
```diff
    function unstakeMunchable(
        uint256 tokenId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        ToilerState memory _toiler = toilerState[tokenId];
        if (_toiler.landlord == address(0)) revert NotStakedError();
        if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();

-        plotOccupied[_toiler.landlord][_toiler.plotId] = Plot({
-            occupied: false,
-            tokenId: 0
-        });
-        toilerState[tokenId] = ToilerState({
-            lastToilDate: 0,
-            plotId: 0,
-            landlord: address(0),
-            latestTaxRate: 0,
-            dirty: false
-        });
-        munchableOwner[tokenId] = address(0);
+       delete plotOccupied[_toiler.landlord][_toiler.plotId];
+       delete toilerState[tokenId];
+       delete munchableOwner[tokenId];
        _removeTokenIdFromStakedList(mainAccount, tokenId);

        munchNFT.transferFrom(address(this), mainAccount, tokenId);
        emit FarmPlotLeave(_toiler.landlord, tokenId, _toiler.plotId);
    }
```
## [N-02] Lack of event emission for configuration updates
Also, there is no event in any internal functions in `_reconfigure()`
```solidity
    function configUpdated() external override onlyConfigStorage {
        _reconfigure();
    }
```

### [NC-03] Same `revert()` checks can be refactored to a modifier
```solidity
    if (plotId >= totalPlotsAvail) revert PlotTooHighError();
    if (plotId >= totalPlotsAvail) revert PlotTooHighError();

    if (_toiler.landlord == address(0)) revert NotStakedError();
    if (_toiler.landlord == address(0)) revert NotStakedError();

    if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();
    if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();
```