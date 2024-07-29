| Severity | Title | 
|:--:|:--:|
| [L-01](#L-01) | Missing setter function to change config storage contract in LandManager| 
| [L-02](#L-02) | Landlord can't force users to farm rewards | 
| [L-03](#L-03) | LandManager::_farmPlots() may try to access an out of bounds element and revert |
| [L-04](#L-04) | LandManager::_farmPlots() may try to access an out of bounds element and revert |
| [L-05](#L-05) | LandManager::transferToUnoccupiedPlot() function unnecessary updates the **toilerState[tokenId].latestTaxRate** |
| [L-06](#L-06) | Missing mechanism in LandManager::_farmPlots() to set the dirty variable to true if landlord plots increase again |

# [L-01] Missing setter function to change config storage contract in LandManager
Currently it is possible to reconfigure the BaseBlastManager state (inherited by LandManager) by using the [configUpdated()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L88-L90) function. But it is not possible to change the config storage contract using the _BaseConfigStoragesetConfigStorage() internal function existing in the BaseBlastManager. It’s unlikely the team might want to change the config contract itself but it would be good to leave the option open in case it ever needs to be changed.

### Recommended Mitigation Steps
Consider adding a restricted function that can change the config storage contract.

# [L-02] Landlord can't force users to farm rewards
Currently only the user that have staked his NFT can claim rewards, and thus only in this way the landlord can receive rewards based on the tax rate he has set. Malicious users can stake in all of the landlord available plots and never farm the rewards by staking again, unstaking, transfering his NFT to another plot, or by simply calling the [farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L228-L230) function. This can be considered a DOS attack, as the landlord won't receive any rewards for his available plots. If non-malicious users have staked in those plots instead, the landlord would have received rewards based on the tax rate he has set. 

### Recommended Mitigation Steps
Consider adding a mechanism where the landlord can force farm rewards for users on some time interval. 

# [L-03] LandManager::_farmPlots() may try to access an out of bounds element and revert
In the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) function, when **finalBonus** is calculated 
```solidity
            finalBonus =
                int16(REALM_BONUSES[(uint256(immutableAttributes.realm) * 5) + uint256(landlordMetadata.snuggeryRealm)]) +
                int16(int8(RARITY_BONUSES[uint256(immutableAttributes.rarity)]));
```
there are no checks if **REALM_BONUSES** or **RARITY_BONUSES** arrays have enough elements, according to the [deployment scripts](https://github.com/code-423n4/2024-07-munchables/blob/main/deployments/utils/consts.ts#L129-L133) **REALM_BONUSES** array has 25 elements and the **RARITY_BONUSES** has 6 elements, however as we can see from the struct and enum definitions in the [MunchablesCommonLib](https://github.com/code-423n4/2024-07-munchables/blob/main/src/libraries/MunchablesCommonLib.sol#L5-L107), the **NFTImmutableAttributes** struct has the **Rarity** and **Realm** enums.
```solidity
    struct NFTImmutableAttributes {
        Rarity rarity;
        uint16 species;
        Realm realm;
        uint8 generation;
        uint32 hatchedDate;
    }
```

The **Rarity** enum has 7 elements and the options in enums are represented by subsequent unsigned integer values starting from 0. So values can vary from **0 to 6**
```solidity
    enum Rarity {
        Primordial,
        Common,
        Rare,
        Epic,
        Legendary,
        Mythic,
        Invalid
    }

```

The **Realm** enum has 6 elements, meaning value can be in the range of **0 to 5**.
```solidity
    enum Realm {
        Everfrost,
        Drench,
        Moltania,
        Arridia,
        Verdentis,
        Invalid
    }
```

The **Player** struct has the **Realm** enum as well:
```solidity
    struct Player {
        uint32 registrationDate;
        uint32 lastPetMunchable;
        uint32 lastHarvestDate;
        Realm snuggeryRealm;
        uint16 maxSnuggerySize;
        uint256 unfedSchnibbles;
        address referrer;
    }
```

If for example the **Realm** element is invalid both for the immutableAttributes and landlordMetadata, or another realm is added, 
```solidity
REALM_BONUSES[(uint256(immutableAttributes.realm) * 5) + uint256(landlordMetadata.snuggeryRealm)]
```
we will get 5 * 5 + 5 = 30, and there is not element at that index

### Recommended Mitigation Steps
When setting up the above described arguments, consider adding some functionality to check if everything will work correctly, and the function call won't revert to  out of bound error.

# [L-04] LandManager::transferToUnoccupiedPlot() function doesn't update the **toilerState.dirty** mapping variable
The [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function allows users to transfer their NFTs to an unoccupied slot, however only the latestTaxRate is updated, which is not necessary as the latestTaxRate is updated in the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) function before the logic in the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function which is called beforehand. However the dirty variable in the **mapping(uint256 => ToilerState) public toilerState;** mapping is not updated
```solidity
    struct ToilerState {
        uint256 lastToilDate;
        uint256 plotId;
        address landlord;
        uint256 latestTaxRate;
        bool dirty;
    }
```
This will be a problem and result in a user loosing rewards in the following scenario. Consider that a user stakes his NFT in the last available plot for a certain landlord, however the landlord decreases his lock in the LockManager contract, and now has less available plots. When the user that staked in the last plot tries to farm his accrued rewards and the [farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L228-L230) function is called which in turn calls the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) internal function, the dirty variable will be set to true. If a user sees that the available plots of a landlord have decreased and directly calls the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function, the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) internal function logic will be executed before the logic in [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) and the dirty variable will be set to true again. If a user transfers his NFT to a previous plot, that should be receiving rewards the dirty variable will still be equal to true, and the user won't be able to accrue and later farm any rewards.

Consider the following example:
 - **LandLordA** has 10 available plots, userA stakes his NFT at plotId 9.
 - **LandLordA** decreases his lock and now only have 5 available plots.
 - **UserA** sees that and decides to move his NFT to plotId 4, which is not occupied, in order to continue receiving rewards. However when he calls the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function, it has the [forceFarmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L40-L43) modifier which will first execute the logic in the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) function before the logic in the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function, which will set the dirty variable to true.
```solidity
            if (_getNumPlots(landlord) < _toiler.plotId) {
                timestamp = plotMetadata[landlord].lastUpdated;
                toilerState[tokenId].dirty = true;
            }
```
Now plotId 4 will be occupied in the **mapping(address => mapping(uint256 => Plot)) plotOccupied;**, but the dirty variable will still be set to true. The next time the user tries to farm his accrued rewards, he won't receive any rewards, the landlord also won't receive the tax rewards he is entitled to. The plotId in the ToilerState is not updated as well, however this is different vulnerability, as a malicious actor can utilize it to DOS all landlord plots. The user here will be thinking that he is acccruing rewards, but when he tries to farm them he won't receive any(if that is his only stake). This results in losses for rewards for both the landlord and the user who first staked, and then transferred his NFT to a different plotId. 

### Recommended Mitigation Steps
When the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function is called set the dirty variable to false, as the function already has logic that checks if the plotId that the user is trying to transfer his NFT, should be receiving rewards or not.
```solidity
if (plotId >= totalPlotsAvail) revert PlotTooHighError();
```

# [L-05] LandManager::transferToUnoccupiedPlot() function unnecessary updates the **toilerState[tokenId].latestTaxRate**
The [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function updates the 
**toilerState[tokenId].latestTaxRate**  to the latest tax set by the owner. However the function has the **forceFarmPlots()** modifier which calls the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) function before the logic in the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function is executed. And as we can see in the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) function:
```solidity
function _farmPlots(address _sender) internal {
        (address mainAccount, MunchablesCommonLib.Player memory renterMetadata) = _getMainAccountRequireRegistered(_sender);

        ...
        for (uint8 i = 0; i < staked.length; i++) {
            ...

            toilerState[tokenId].lastToilDate = timestamp;
            toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord].currentTaxRate;

            ...
        }
        ...;
    }
```
The **latestTaxRate** is already updated.
### Recommended Mitigation Steps 
Remove the following line from the [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function
```solidity
toilerState[tokenId].latestTaxRate = plotMetadata[_toiler.landlord].currentTaxRate;
```
# [L-06] Missing mechanism in LandManager::_farmPlots() to set the dirty variable to true if landlord plots increase again
In the [_farmPlots()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L232-L310) if a user has staked lets say in the last available **plotId**, but the landlord available plots decrease due to the landlord decreasing his locked tokens in the LockManager contract, and the user farms his accrued rewards, the dirty variable for his NFT id will be set to true.
```solidity
            if (_getNumPlots(landlord) < _toiler.plotId) {
                timestamp = plotMetadata[landlord].lastUpdated;
                toilerState[tokenId].dirty = true;
            }
```
However if the user hasn't unstaked and the landlord available plots increase, due to the landlord staking more in his lock, or the **PRICE_PER_PLOT** decreasing, and a user tries to farm rewards he won't be able to do so for the NFT id that has the dirty variable set to true. The user can unstake and stake again in his previous plot. However, if he is not aware that the dirty variable for his NFT didn't change back to false, and waits for 2 weeks to try and farm rewards, and see that he didn't receive any rewards, he will effectively loose the rewards, that he should have accrued during those 2 weeks.

### Recommended Mitigation Steps 
Consider adding a check, if a user has had his dirty variable set to true previously, and now the plotId he staked in should receive rewards again, set the dirty variable to false again. That fact that users may not farm rewards that offten should be taken into consideration as well, as to when the dirty variable is update to false again. 