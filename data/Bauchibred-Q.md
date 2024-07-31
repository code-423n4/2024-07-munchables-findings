# QA Report

## Table of Contents

| Issue ID | Description |
| -------- | ----------- |
| [QA-01](#qa-01-consider-not-having-a-jarring-dependence-on-not-allowing-staking-to-a-particular-address) | Consider not having a jarring dependence on not allowing staking to a particular address |
| [QA-02](#qa-02-consider-having-variables-for-max-_invariants_) | Consider having variables for max _invariants_ |
| [QA-03](#qa-03-consider-introducing-some-sort-of-documentation/code-comments) | Consider introducing some sort of documentation/code comments |
| [QA-04](#qa-04-users-could-be-blocked-off-from-staking-for-a-long-time-due-to-an-admin-action) | Users could be blocked off from staking for a long time due to an admin action |
| [QA-05](#qa-05-import-declarations-should-import-specific-identifiers-rather-than-the-whole-file) | Import declarations should import specific identifiers, rather than the whole file |
| [QA-06](#qa-06-consider-making-the--if-(munchablesstaked[mainaccount].length->-10)-check-inclusive) | Consider making the ` if (munchablesStaked[mainAccount].length > 10)` check inclusive |
| [QA-07](#qa-07-suggested-alternative-method-_farmplots()-in-regards-its-current-edge-case-with-the-plots-updated-timestamps) | Suggested alternative method `_farmPlots()` in regards it's current edge case with the plots updated timestamps |
| [QA-08](#qa-08-consider-attaching-more-test-cases) | Consider attaching more test cases |

## QA-01 Consider not having a jarring dependence on not allowing staking to a particular address

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L171

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }
```

This function is used to stake a munchable NFT and there exists an invariant currently that the one is never allowed to stake to oneself, however this can't really be tracked cause nothing stops the current owner of the nft to just stake to another account of his, so best would be to actually create the staking logic without having any particular non-acceptable logic for the address to which one can stake to.

### Impact

QA, design improvement.

### Recommended Mitigation Steps

Consider not having a heavy dependence on the `who` the tokens get staked to.

## QA-02 Consider having variables for max _invariants_

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L171

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }
```

This function is used to stake a munchable NFT and there exists an invariant currently that one is never allowed to have more than `10` stakes at a time, this however can be further made by introducing a `maxStakesAllowed` variable and then the check is done on this variable,

### Impact

QA, additionally with this approach we can then introduce an admin backed function that could update the max accepted stakes at a time.

### Recommended Mitigation Steps

Consider applying these changes:

```diff
..
+ uint8 maxStakesAllowed;

..

+    function setmaxStakesAllowed(uint256 _maxStakesAllowed) onlyOwner{
+        maxStakesAllowed = _maxStakesAllowed;

+    }

..
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
-        if (munchablesStaked[mainAccount].length > 10)
+        if (munchablesStaked[mainAccount].length > maxStakesAllowed)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }


```

## QA-03 Consider introducing some sort of documentation/code comments

### Proof of Concept

Going through the only in scope contract _([LandManager.sol](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol))_ we would see that most if not all functionalities lack any documentation or code comments whatsoever, for e.g take a look at these three functions: https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L227

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }

    function unstakeMunchable(
        uint256 tokenId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        ToilerState memory _toiler = toilerState[tokenId];
        if (_toiler.landlord == address(0)) revert NotStakedError();
        if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();

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
        munchableOwner[tokenId] = address(0);
        _removeTokenIdFromStakedList(mainAccount, tokenId);

        munchNFT.transferFrom(address(this), mainAccount, tokenId);
        emit FarmPlotLeave(_toiler.landlord, tokenId, _toiler.plotId);
    }

    function transferToUnoccupiedPlot(
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        ToilerState memory _toiler = toilerState[tokenId];
        uint256 oldPlotId = _toiler.plotId;
        uint256 totalPlotsAvail = _getNumPlots(_toiler.landlord);
        if (_toiler.landlord == address(0)) revert NotStakedError();
        if (munchableOwner[tokenId] != mainAccount) revert InvalidOwnerError();
        if (plotOccupied[_toiler.landlord][plotId].occupied)
            revert OccupiedPlotError(_toiler.landlord, plotId);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

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

No comment whatsoever and neither are they explained in an interface like file.

### Impact

QA

### Recommended Mitigation Steps

Consider introducing heavy documentation/code comments

## QA-04 Users could be blocked off from staking for a long time due to an admin action

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L172

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }

```

This function is used to stake a Munchable NFT, evidently, it includes a check that the plot is not too high CC https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L344-L347

```solidity
    function _getNumPlots(address _account) internal view returns (uint256) {
        return lockManager.getLockedWeightedValue(_account) / PRICE_PER_PLOT;
    }
}
```

Issue however is that the `PRICE_PER_PLOT` can be set to any value by the admins, which then mean that a user can effectively be made to be unstakable for as long as possible

### Impact

QA

> Centralisation risk, but per C4's new QA update, this should be flagged in an audit.

### Recommended Mitigation Steps

N/A

## QA-05 Import declarations should import specific identifiers, rather than the whole file

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L4-L9

```solidity
import "../interfaces/ILandManager.sol";
import "../interfaces/ILockManager.sol";
import "../interfaces/IAccountManager.sol";
import "./BaseBlastManagerUpgradeable.sol";
import "../interfaces/INFTAttributesManager.sol";
import "openzeppelin-contracts/contracts/token/ERC721/IERC721.sol";
```

Evidently, the imports being done is not name specific, but this is not the best implementation cause this could lead to polluting the symbol namespace.

### Impact

QA, albeit this could lead to the potential pollution of the symbol namespace and a slower compilation speed.

### Recommended Mitigation Steps

Consider using import declarations of the form `import {<identifier_name>} from "some/file.sol"` which avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation (but does not save any gas)

## QA-06 Consider making the ` if (munchablesStaked[mainAccount].length > 10)` check inclusive

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L171

```solidity
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
        if (munchablesStaked[mainAccount].length > 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }
```

This function is used to stake a munchable NFT and there exists an invariant currently that one is never allowed to have more than `10` stakes at a time, this however is not sufficiently protected against and a staker can have more than `10` stakes cause the check is not inclusive.

### Impact

Depends on the intended functionality, however this screams a broken invariant, considering an integrator could stake more than the intended `max` per the documentation/intended functionality.

### Recommended Mitigation Steps

Consider applying these changes

```diff
    function stakeMunchable(
        address landlord,
        uint256 tokenId,
        uint256 plotId
    ) external override forceFarmPlots(msg.sender) notPaused {
        (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
        if (landlord == mainAccount) revert CantStakeToSelfError();
        if (plotOccupied[landlord][plotId].occupied)
            revert OccupiedPlotError(landlord, plotId);
-        if (munchablesStaked[mainAccount].length > 10)
+        if (munchablesStaked[mainAccount].length >= 10)
            revert TooManyStakedMunchiesError();
        if (munchNFT.ownerOf(tokenId) != mainAccount)
            revert InvalidOwnerError();

        uint256 totalPlotsAvail = _getNumPlots(landlord);
        if (plotId >= totalPlotsAvail) revert PlotTooHighError();

        if (
            !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
            munchNFT.getApproved(tokenId) != address(this)
        ) revert NotApprovedError();
        munchNFT.transferFrom(mainAccount, address(this), tokenId);

        plotOccupied[landlord][plotId] = Plot({
            occupied: true,
            tokenId: tokenId
        });

        munchablesStaked[mainAccount].push(tokenId);
        munchableOwner[tokenId] = mainAccount;

        toilerState[tokenId] = ToilerState({
            lastToilDate: block.timestamp,
            plotId: plotId,
            landlord: landlord,
            latestTaxRate: plotMetadata[landlord].currentTaxRate,
            dirty: false
        });

        emit FarmPlotTaken(toilerState[tokenId], tokenId);
    }
```

> NB: After seeing this issue to be a borderline `1/2` on C4, a more detailed report with a coded POC has been submitted with a `2` rating

## QA-07 Suggested alternative method `_farmPlots()` in regards it's current edge case with the plots updated timestamps

### Proof of Concept

First see https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/README.md#L19-L20

> Within the `_farmPlots()` function, the following edge case can happen:
> If the plot ID is lower than the max plot amount, the system uses the last updated plot metadata time and tracks this with a dirty boolean flag. The edge case occurs when a user hasn’t farmed in a while and the landlord updates the plots multiple times (ie unlocks/locks multiple times), causing the last updated time to reflect the latest lock update rather than the first. This is a known edge case that we are comfortable with. HOWEVER, we will accept alternative solutions to mitigate this issue. Pointing it out will not result in a valid finding.

Now see the current implementation of the `_farmPlots()` function here https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L232-L310

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
            (
                ,
                MunchablesCommonLib.Player memory landlordMetadata
            ) = _getMainAccountRequireRegistered(landlord);

            immutableAttributes = nftAttributesManager.getImmutableAttributes(
                tokenId
            );
            finalBonus =
                int16(
                    REALM_BONUSES[
                        (uint256(immutableAttributes.realm) * 5) +
                            uint256(landlordMetadata.snuggeryRealm)
                    ]
                ) +
                int16(
                    int8(RARITY_BONUSES[uint256(immutableAttributes.rarity)])
                );
            schnibblesTotal =
                (timestamp - _toiler.lastToilDate) *
                BASE_SCHNIBBLE_RATE;
            schnibblesTotal = uint256(
                (int256(schnibblesTotal) +
                    (int256(schnibblesTotal) * finalBonus)) / 100
            );
            schnibblesLandlord =
                (schnibblesTotal * _toiler.latestTaxRate) /
                1e18;

            toilerState[tokenId].lastToilDate = timestamp;
            toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
                .currentTaxRate;

            renterMetadata.unfedSchnibbles += (schnibblesTotal -
                schnibblesLandlord);

            landlordMetadata.unfedSchnibbles += schnibblesLandlord;
            landlordMetadata.lastPetMunchable = uint32(timestamp);
            accountManager.updatePlayer(landlord, landlordMetadata);
            emit FarmedSchnibbles(
                _toiler.landlord,
                tokenId,
                _toiler.plotId,
                schnibblesTotal - schnibblesLandlord,
                schnibblesLandlord
            );
        }
        accountManager.updatePlayer(mainAccount, renterMetadata);
    }
```

To address the edge case where the last updated plot metadata time reflects the latest lock update rather than the first, we can try to implement an alternative solution that tracks multiple updates and calculates the correct farming rewards based on the historical data. The approach in this report is going to include using a history of the plot updates.

So we first introduce a new data structure to store the history of plot updates for each landlord. This can be a mapping from landlord addresses to an array of timestamps representing the times at which the plots were updated. And then modify the `_farmPlots` function to use the plot update history to calculate the correct farming rewards.

### Impact

QA, providing a requested walk around for the current edge case from the readMe here.

### Recommended Mitigation Steps

1. **Data Structure for Plot Update History:**

   - We introduce a new mapping `plotUpdateHistory` to store the history of plot updates for each landlord. This mapping stores an array of timestamps representing the times at which the plots were updated.

2. **Updating Plot Update History:**

   - In the `_reconfigure` function, we add the current timestamp to the plot update history whenever the plots are updated. This ensures that we maintain a record of all plot updates.

3. **Using Plot Update History in `_farmPlots`:**
   - In the `_farmPlots` function, we retrieve the plot update history for the landlord and use it to calculate the correct farming rewards. We iterate through the update history to find the last relevant update that occurred after the last toil date and before the current timestamp. This ensures that we calculate the farming rewards based on the correct time period.

By implementing this alternative solution, we can mitigate the edge case where the last updated plot metadata time reflects the latest lock update rather than the first. This approach ensures that the farming rewards are calculated accurately based on the historical data of plot updates.

Pseudo fixes to consider:

Add a new mapping to store the history of plot updates:

```diff
// landlord to array of plot update timestamps
+ mapping(address => uint256[]) plotUpdateHistory;


//Ensure that the plot update history is maintained whenever the plots are updated:

function _reconfigure() internal {
    // existing code...

    // Add the current timestamp to the plot update history
+    plotUpdateHistory[landlord].push(block.timestamp);

    // existing code...
}
// Update the `_farmPlots` function to use the plot update history:


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

        // Use the plot update history to calculate the correct farming rewards
+        uint256[] memory updateHistory = plotUpdateHistory[landlord];
        uint256 lastRelevantUpdate = _toiler.lastToilDate;
        for (uint256 j = 0; j < updateHistory.length; j++) {
            if (updateHistory[j] > _toiler.lastToilDate && updateHistory[j] < timestamp) {
                lastRelevantUpdate = updateHistory[j];
                break;
            }
        }

        // Calculate the farming rewards based on the last relevant update
        schnibblesTotal =
            (timestamp - lastRelevantUpdate) *
            BASE_SCHNIBBLE_RATE;
        schnibblesTotal = uint256(
            (int256(schnibblesTotal) +
                (int256(schnibblesTotal) * finalBonus)) / 100
        );
        schnibblesLandlord =
            (schnibblesTotal * _toiler.latestTaxRate) /
            1e18;

        toilerState[tokenId].lastToilDate = timestamp;
        toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord]
            .currentTaxRate;

        renterMetadata.unfedSchnibbles += (schnibblesTotal -
            schnibblesLandlord);

        landlordMetadata.unfedSchnibbles += schnibblesLandlord;
        landlordMetadata.lastPetMunchable = uint32(timestamp);
        accountManager.updatePlayer(landlord, landlordMetadata);
        emit FarmedSchnibbles(
            _toiler.landlord,
            tokenId,
            _toiler.plotId,
            schnibblesTotal - schnibblesLandlord,
            schnibblesLandlord
        );
    }
    accountManager.updatePlayer(mainAccount, renterMetadata);
}
```

### Recommended Mitigation Steps

## QA-08 Consider attaching more test cases

### Proof of Concept

Protocol currently has different test files for different core functionalities, including staking/unstaking from the land manager, issue however is that these test coverages are not high enough.

For eg, consider https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/tests/managers/LandManager/stakeMunchable.test.ts

Evidently some tests are missing, to hint one, there is no test that when the contract is paused the unstaking attempts revert.

### Impact

info

### Recommended Mitigation Steps

Consider attaching more test cases.