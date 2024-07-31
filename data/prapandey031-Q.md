# Low Risk Issues
----------------------------------


## [LOW-1] No ```notPaused``` modifier in the ```updatePlotMetadata(address landlord)``` function: 

### Description
The ```updatePlotMetadata(address landlord)``` function sets updates the plot metadata of the landlords. However, it should not be done if the LandManager contract is paused.

### Instances
There is only 1 instance of this issue.

LOC-116:

```solidity
    function updatePlotMetadata(
        address landlord
    ) external override onlyConfiguredContract(StorageKey.AccountManager) {
```

### Recommended Mitigation:

Add the ```notPaused``` modifier in the ```updatePlotMetadata(address landlord)``` function.


## [LOW-2] ```plotMetadata[landlord].lastUpdated``` is not updated in the ```updateTaxRate(uint256 newTaxRate)``` function:

### Description
Whenever the metadata of a landlord is updated, its ```plotMetadata[landlord].lastUpdated``` is also updated with ```block.timestamp```. However, it is not done so in the ```updateTaxRate(uint256 newTaxRate)``` function.

### Instances
There is only 1 instance of this issue.

Loc-92:

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

### Recommended Mitigation:
Add the following statement in the ```updateTaxRate(uint256 newTaxRate)``` function:

```solidity
plotMetadata[landlord].lastUpdated = block.timestamp;
```


## [LOW-3] Use a mapping to save going through a loop in ```_removeTokenIdFromStakedList()``` function:

### Description
The loop exists in the ```_removeTokenIdFromStakedList()``` function to check the existence of ```tokenId``` in the staked array. However, there can be a mapping from tokenId to (staked array index + 1), and, this mapping could be used to check for the existence of a tokenId without going through a loop.

### Instances

LOC-312:

```solidity
    function _removeTokenIdFromStakedList(
        address mainAccount,
        uint256 tokenId
    ) internal {
        uint256 stakedLength = munchablesStaked[mainAccount].length;
        bool found = false;
        for (uint256 i = 0; i < stakedLength; i++) {
            if (munchablesStaked[mainAccount][i] == tokenId) {
                munchablesStaked[mainAccount][i] = munchablesStaked[
                    mainAccount
                ][stakedLength - 1];
                found = true;
                munchablesStaked[mainAccount].pop();
                break;
            }
        }

        if (!found) revert InvalidTokenIdError();
    }
```

### Recommended Mitigation
Define a mapping:

```solidity
mapping(address => mapping(uint256 => uint256)) tokIdToStakeIndex;
```

And search for the existence of a token Id using:

```solidity
if (tokIdToStakeIndex[owner][tokenId] == 0) revert tokenIdDoesNotExistError();

else { .....}
```

The mapping would contain (index + 1) to account for indices starting from 0.


## [LOW-4] No check of ```plotMetadata[landlord].lastUpdated == 0``` in unstake and transfer functions:

### Description
The ```unstakeMunchable(uint256 tokenId)``` and the ```transferToUnoccupiedPlot()``` functions do not check for ```plotMetadata[landlord].lastUpdated``` to be non-zero.

### Recommended Mitigation
Add the following in both the unstake and transfer functions:

```solidity
        if (plotMetadata[mainAccount].lastUpdated == 0)
            revert();
```

## [LOW-5] No comments in the LandManager contract:

### Description
No sufficient comment is provided for any function in the LandManager contract.

### Recommended Mitigation
Add comments in the LandManager contract


## [LOW-6] Variables should use camelCase:

### Description
Variables should use camelCase naming convention

### Instances 

Total 7 instances:

```solidity
    uint256 MIN_TAX_RATE;
    uint256 MAX_TAX_RATE;
    uint256 DEFAULT_TAX_RATE;
    uint256 BASE_SCHNIBBLE_RATE;
    uint256 PRICE_PER_PLOT;
    int16[] REALM_BONUSES;
    uint8[] RARITY_BONUSES;
```

### Recommended Mitigation
Use camelCase for all the mentioned variables