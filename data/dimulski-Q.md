| Severity | Title | 
|:--:|:--:|
| [L-01](#L-01) | Missing setter function to change config storage contract in LandManager| 
| [L-02](#L-02) | Landlord can't force users to farm rewards | 
| [L-03](#L-03) | LandManager::_farmPlots() may try to access an out of bounds element and revert |
| [L-03](#L-03) | LandManager::_farmPlots() may try to access an out of bounds element and revert |
| [L-03](#L-03) | LandManager::_farmPlots() may try to access an out of bounds element and revert |

# [L-01] Missing setter function to change config storage contract in LandManager
Currently it is possible to reconfigure the BaseBlastManager state (inherited by LandManager) by using the [configUpdated()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L88-L90) function. But it is not possible to change the config storage contract using the _BaseConfigStoragesetConfigStorage() internal function existing in the BaseBlastManager.

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
### Recommended Mitigation Steps

# [L-04] LandManager::transferToUnoccupiedPlot() function doesn't update the **toilerState.dirty** mapping variable
The [transferToUnoccupiedPlot()](https://github.com/code-423n4/2024-07-munchables/blob/main/src/managers/LandManager.sol#L199-L226) function allows users to transfer their NFTs to an unoccupied slot, however only the tax 
### Recommended Mitigation Steps

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
# [L-06] Missing mechanism in LandManager::_farmPlots()
### Recommended Mitigation Steps 