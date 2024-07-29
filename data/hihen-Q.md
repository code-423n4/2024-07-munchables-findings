# QA Report

Issues that are not included in Automated Findings.

## Summary

### Low Issues

Total **13 instances** over **6 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[L-01]](#l-01-missing-zero-address-check-in-initializer) | Missing zero address check in initializer | 1 |
| [[L-02]](#l-02-constructor--initialization-function-lacks-parameter-validation) | Constructor / initialization function lacks parameter validation | 1 |
| [[L-03]](#l-03-functions-calling-contractsaddresses-with-transfer-hooks-should-be-protected-by-reentrancy-guard) | Functions calling contracts/addresses with transfer hooks should be protected by reentrancy guard | 2 |
| [[L-04]](#l-04-critical-functions-should-be-controlled-by-time-locks) | Critical functions should be controlled by time locks | 2 |
| [[L-05]](#l-05-the-feesrates-should-be-capped-by-smart-contracts) | The fees/rates should be capped by smart contracts | 4 |
| [[L-06]](#l-06-code-does-not-follow-the-best-practice-of-check-effects-interaction) | Code does not follow the best practice of check-effects-interaction | 3 |

### Non Critical Issues

Total **123 instances** over **29 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[N-01]](#n-01-import-declarations-should-import-specific-identifiers-rather-than-the-whole-file) | Import declarations should import specific identifiers, rather than the whole file | 6 |
| [[N-02]](#n-02-visibility-of-state-variables-is-not-explicitly-defined) | Visibility of state variables is not explicitly defined | 16 |
| [[N-03]](#n-03-names-of-privateinternal-state-variables-should-be-prefixed-with-an-underscore) | Names of `private`/`internal` state variables should be prefixed with an underscore | 16 |
| [[N-04]](#n-04-use-of-override-is-unnecessary) | Use of `override` is unnecessary | 8 |
| [[N-05]](#n-05-consider-providing-a-ranged-getter-for-array-state-variables) | Consider providing a ranged getter for array state variables | 3 |
| [[N-06]](#n-06-consider-splitting-complex-checks-into-multiple-steps) | Consider splitting complex checks into multiple steps | 2 |
| [[N-07]](#n-07-complex-casting) | Complex casting | 3 |
| [[N-08]](#n-08-consider-adding-a-blockdeny-list) | Consider adding a block/deny-list | 1 |
| [[N-09]](#n-09-upper_case-names-should-be-reserved-for-constantimmutable-variables) | UPPER_CASE names should be reserved for `constant`/`immutable` variables | 7 |
| [[N-10]](#n-10-consider-emitting-an-event-at-the-end-of-the-constructor) | Consider emitting an event at the end of the constructor | 1 |
| [[N-11]](#n-11-events-are-emitted-without-the-sender-information) | Events are emitted without the sender information | 7 |
| [[N-12]](#n-12-imports-could-be-organized-more-systematically) | Imports could be organized more systematically | 1 |
| [[N-13]](#n-13-there-is-no-need-to-initialize-bool-variables-with-false) | There is no need to initialize bool variables with `false` | 1 |
| [[N-14]](#n-14-named-imports-of-parent-contracts-are-missing) | Named imports of parent contracts are missing | 2 |
| [[N-15]](#n-15-constants-should-be-put-on-the-left-side-of-comparisons) | Constants should be put on the left side of comparisons | 6 |
| [[N-16]](#n-16-high-cyclomatic-complexity) | High cyclomatic complexity | 1 |
| [[N-17]](#n-17-unnecessary-casting) | Unnecessary casting | 9 |
| [[N-18]](#n-18-consider-using-delete-rather-than-assigning-zero-to-clear-values) | Consider using `delete` rather than assigning zero to clear values | 1 |
| [[N-19]](#n-19-consider-using-named-function-arguments) | Consider using named function arguments | 2 |
| [[N-20]](#n-20-named-returns-are-recommended) | Named returns are recommended | 2 |
| [[N-21]](#n-21-returning-a-struct-instead-of-a-bunch-of-variables-is-better) | Returning a struct instead of a bunch of variables is better | 1 |
| [[N-22]](#n-22-contract-variables-should-have-comments) | Contract variables should have comments | 11 |
| [[N-23]](#n-23-multiple-mappings-with-same-keys-can-be-combined-into-a-single-struct-mapping-for-readability) | Multiple mappings with same keys can be combined into a single struct mapping for readability | 5 |
| [[N-24]](#n-24-missing-event-for-critical-changes) | Missing event for critical changes | 1 |
| [[N-25]](#n-25-consider-adding-emergency-stop-functionality) | Consider adding emergency-stop functionality | 1 |
| [[N-26]](#n-26-missing-checks-for-uint-state-variable-assignments) | Missing checks for uint state variable assignments | 5 |
| [[N-27]](#n-27-large-or-complicated-code-bases-should-implement-invariant-tests) | Large or complicated code bases should implement invariant tests | 1 |
| [[N-28]](#n-28-the-default-value-is-manually-set-when-it-is-declared) | The default value is manually set when it is declared | 2 |
| [[N-29]](#n-29-contracts-should-have-all-publicexternal-functions-exposed-by-interfaces) | Contracts should have all `public`/`external` functions exposed by `interface`s | 1 |

## Low Issues

### [L-01] Missing zero address check in initializer

Consider adding a zero address check for each address type parameter in initializer.

There is 1 instance:

- *LandManager.sol* ( [45-45](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L45-L45) ):

```solidity
/// @audit `_configStorage not checked`
45:     function initialize(address _configStorage) public override initializer {
```

### [L-02] Constructor / initialization function lacks parameter validation

Constructors and initialization functions play a critical role in contracts by setting important initial states when the contract is first deployed before the system starts. The parameters passed to the constructor and initialization functions directly affect the behavior of the contract / protocol. If incorrect parameters are provided, the system may fail to run, behave abnormally, be unstable, or lack security. Therefore, it's crucial to carefully check each parameter in the constructor and initialization functions. If an exception is found, the transaction should be rolled back.

There is 1 instance:

- *LandManager.sol* ( [45-45](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L45-L45) ):

```solidity
/// @audit `_configStorage`
45:     function initialize(address _configStorage) public override initializer {
```

### [L-03] Functions calling contracts/addresses with transfer hooks should be protected by reentrancy guard

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks opens the users of this protocol up to [read-only reentrancy vulnerability](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect them except by block-listing the entire protocol.

There are 2 instances:

- *LandManager.sol* ( [152-152](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L152-L152), [195-195](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L195-L195) ):

```solidity
152:         munchNFT.transferFrom(mainAccount, address(this), tokenId);

195:         munchNFT.transferFrom(address(this), mainAccount, tokenId);
```

### [L-04] Critical functions should be controlled by time locks

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

There are 2 instances:

- *LandManager.sol* ( [116-118](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L116-L118), [312-315](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L312-L315) ):

```solidity
116:     function updatePlotMetadata(
117:         address landlord
118:     ) external override onlyConfiguredContract(StorageKey.AccountManager) {

312:     function _removeTokenIdFromStakedList(
313:         address mainAccount,
314:         uint256 tokenId
315:     ) internal {
```

### [L-05] The fees/rates should be capped by smart contracts

The fees/rates should be required to be less than 100%, preferably at a much lower limit, so that users don't have to monitor the blockchain for changes when using the protocol.

There are 4 instances:

- *LandManager.sol* ( [65-67](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L65-L67), [68-70](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L68-L70), [71-73](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L71-L73), [74-76](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L74-L76) ):

```solidity
65:         MIN_TAX_RATE = IConfigStorage(configStorage).getUint(
66:             StorageKey.LockManager
67:         );

68:         MAX_TAX_RATE = IConfigStorage(configStorage).getUint(
69:             StorageKey.AccountManager
70:         );

71:         DEFAULT_TAX_RATE = IConfigStorage(configStorage).getUint(
72:             StorageKey.ClaimManager
73:         );

74:         BASE_SCHNIBBLE_RATE = IConfigStorage(configStorage).getUint(
75:             StorageKey.MigrationManager
76:         );
```

### [L-06] Code does not follow the best practice of check-effects-interaction

Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

There are 3 instances:

- *LandManager.sol* ( [154-157](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L154-L157), [160-160](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L160-L160), [162-168](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L162-L168) ):

```solidity
/// @audit `_getMainAccountRequireRegistered()` is called on line 136, triggering an external call - `getPlayer()`
154:         plotOccupied[landlord][plotId] = Plot({
155:             occupied: true,
156:             tokenId: tokenId
157:         });

/// @audit `_getMainAccountRequireRegistered()` is called on line 136, triggering an external call - `getPlayer()`
160:         munchableOwner[tokenId] = mainAccount;

/// @audit `_getMainAccountRequireRegistered()` is called on line 136, triggering an external call - `getPlayer()`
162:         toilerState[tokenId] = ToilerState({
163:             lastToilDate: block.timestamp,
164:             plotId: plotId,
165:             landlord: landlord,
166:             latestTaxRate: plotMetadata[landlord].currentTaxRate,
167:             dirty: false
168:         });
```

## Non Critical Issues

### [N-01] Import declarations should import specific identifiers, rather than the whole file

Using import declarations of the form `import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation (but does not save any gas).

There are 6 instances:

- *LandManager.sol* ( [4](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L4), [5](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L5), [6](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L6), [7](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L7), [8](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L8), [9](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L9) ):

```solidity
4: import "../interfaces/ILandManager.sol";

5: import "../interfaces/ILockManager.sol";

6: import "../interfaces/IAccountManager.sol";

7: import "./BaseBlastManagerUpgradeable.sol";

8: import "../interfaces/INFTAttributesManager.sol";

9: import "openzeppelin-contracts/contracts/token/ERC721/IERC721.sol";
```

### [N-02] Visibility of state variables is not explicitly defined

To avoid misunderstandings and unexpected state accesses, it is recommended to explicitly define the visibility of each state variable.

<details>
<summary>There are 16 instances (click to show):</summary>

- *LandManager.sol* ( [12-12](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L12-L12), [13-13](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L13-L13), [14-14](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L14-L14), [15-15](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L15-L15), [16-16](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L16-L16), [17-17](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L17-L17), [18-18](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L18-L18), [21-21](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L21-L21), [23-23](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L23-L23), [25-25](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L25-L25), [27-27](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L27-L27), [29-29](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L29-L29), [31-31](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L31-L31), [32-32](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L32-L32), [33-33](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L33-L33), [34-34](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L34-L34) ):

```solidity
12:     uint256 MIN_TAX_RATE;

13:     uint256 MAX_TAX_RATE;

14:     uint256 DEFAULT_TAX_RATE;

15:     uint256 BASE_SCHNIBBLE_RATE;

16:     uint256 PRICE_PER_PLOT;

17:     int16[] REALM_BONUSES;

18:     uint8[] RARITY_BONUSES;

21:     mapping(address => PlotMetadata) plotMetadata;

23:     mapping(address => mapping(uint256 => Plot)) plotOccupied;

25:     mapping(uint256 => address) munchableOwner;

27:     mapping(address => uint256[]) munchablesStaked;

29:     mapping(uint256 => ToilerState) toilerState;

31:     ILockManager lockManager;

32:     IAccountManager accountManager;

33:     IERC721 munchNFT;

34:     INFTAttributesManager nftAttributesManager;
```

</details>

### [N-03] Names of `private`/`internal` state variables should be prefixed with an underscore

It is recommended by the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#underscore-prefix-for-non-external-functions-and-variables).

<details>
<summary>There are 16 instances (click to show):</summary>

- *LandManager.sol* ( [12-12](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L12-L12), [13-13](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L13-L13), [14-14](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L14-L14), [15-15](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L15-L15), [16-16](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L16-L16), [17-17](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L17-L17), [18-18](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L18-L18), [21-21](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L21-L21), [23-23](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L23-L23), [25-25](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L25-L25), [27-27](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L27-L27), [29-29](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L29-L29), [31-31](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L31-L31), [32-32](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L32-L32), [33-33](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L33-L33), [34-34](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L34-L34) ):

```solidity
12:     uint256 MIN_TAX_RATE;

13:     uint256 MAX_TAX_RATE;

14:     uint256 DEFAULT_TAX_RATE;

15:     uint256 BASE_SCHNIBBLE_RATE;

16:     uint256 PRICE_PER_PLOT;

17:     int16[] REALM_BONUSES;

18:     uint8[] RARITY_BONUSES;

21:     mapping(address => PlotMetadata) plotMetadata;

23:     mapping(address => mapping(uint256 => Plot)) plotOccupied;

25:     mapping(uint256 => address) munchableOwner;

27:     mapping(address => uint256[]) munchablesStaked;

29:     mapping(uint256 => ToilerState) toilerState;

31:     ILockManager lockManager;

32:     IAccountManager accountManager;

33:     IERC721 munchNFT;

34:     INFTAttributesManager nftAttributesManager;
```

</details>

### [N-04] Use of `override` is unnecessary

[Starting from Solidity 0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), the override keyword is not required when overriding an interface function, except for the case where the function is defined in multiple bases.

There are 8 instances:

- *LandManager.sol* ( [88-88](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L88-L88), [92-92](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L92-L92), [104-104](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L104-L104), [116-118](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L116-L118), [131-135](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L135), [173-175](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L173-L175), [199-202](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L199-L202), [228-228](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L228-L228) ):

```solidity
88:     function configUpdated() external override onlyConfigStorage {

92:     function updateTaxRate(uint256 newTaxRate) external override notPaused {

104:     function triggerPlotMetadata() external override notPaused {

116:     function updatePlotMetadata(
117:         address landlord
118:     ) external override onlyConfiguredContract(StorageKey.AccountManager) {

131:     function stakeMunchable(
132:         address landlord,
133:         uint256 tokenId,
134:         uint256 plotId
135:     ) external override forceFarmPlots(msg.sender) notPaused {

173:     function unstakeMunchable(
174:         uint256 tokenId
175:     ) external override forceFarmPlots(msg.sender) notPaused {

199:     function transferToUnoccupiedPlot(
200:         uint256 tokenId,
201:         uint256 plotId
202:     ) external override forceFarmPlots(msg.sender) notPaused {

228:     function farmPlots() external override notPaused {
```

### [N-05] Consider providing a ranged getter for array state variables

While the compiler automatically provides a getter for accessing single elements within a public state variable array, it doesn't provide a way to fetch the whole array, or subsets thereof. Consider adding a function to allow the fetching of slices of the array, especially if the contract doesn't already have multicall functionality.

There are 3 instances:

- *LandManager.sol* ( [17-17](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L17-L17), [18-18](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L18-L18), [27-27](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L27-L27) ):

```solidity
17:     int16[] REALM_BONUSES;

18:     uint8[] RARITY_BONUSES;

27:     mapping(address => uint256[]) munchablesStaked;
```

### [N-06] Consider splitting complex checks into multiple steps

Assign the expression's parts to intermediate local variables, and check against those instead.

There are 2 instances:

- *LandManager.sol* ( [94-94](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L94-L94), [149-150](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L149-L150) ):

```solidity
94:         if (newTaxRate < MIN_TAX_RATE || newTaxRate > MAX_TAX_RATE)

149:             !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
150:             munchNFT.getApproved(tokenId) != address(this)
```

### [N-07] Complex casting

Consider whether the number of casts is really necessary, or whether using a different type would be more appropriate. Alternatively, add comments to explain in detail why the casts are necessary, and any implicit reasons why the cast does not introduce an overflow.

There are 3 instances:

- *LandManager.sol* ( [271-276](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L271-L276), [277-279](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L277-L279), [283-286](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L283-L286) ):

```solidity
271:                 int16(
272:                     REALM_BONUSES[
273:                         (uint256(immutableAttributes.realm) * 5) +
274:                             uint256(landlordMetadata.snuggeryRealm)
275:                     ]
276:                 ) +

277:                 int16(
278:                     int8(RARITY_BONUSES[uint256(immutableAttributes.rarity)])
279:                 );

283:             schnibblesTotal = uint256(
284:                 (int256(schnibblesTotal) +
285:                     (int256(schnibblesTotal) * finalBonus)) / 100
286:             );
```

### [N-08] Consider adding a block/deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens

There is 1 instance:

- *LandManager.sol* ( [11](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L11) ):

```solidity
11: contract LandManager is BaseBlastManagerUpgradeable, ILandManager {
```

### [N-09] UPPER_CASE names should be reserved for `constant`/`immutable` variables

If the variable needs to be different based on which class it comes from, a `view`/`pure` _function_ should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).

There are 7 instances:

- *LandManager.sol* ( [12](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L12), [13](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L13), [14](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L14), [15](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L15), [16](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L16), [17](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L17), [18](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L18) ):

```solidity
12:     uint256 MIN_TAX_RATE;

13:     uint256 MAX_TAX_RATE;

14:     uint256 DEFAULT_TAX_RATE;

15:     uint256 BASE_SCHNIBBLE_RATE;

16:     uint256 PRICE_PER_PLOT;

17:     int16[] REALM_BONUSES;

18:     uint8[] RARITY_BONUSES;
```

### [N-10] Consider emitting an event at the end of the constructor

This will allow users to easily exactly pinpoint when and by whom a contract was constructed.

There is 1 instance:

- *LandManager.sol* ( [36-36](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L36-L36) ):

```solidity
36:     constructor() {
```

### [N-11] Events are emitted without the sender information

When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the `msg.sender` the events of these types of action will make events much more useful to end users, especially when `msg.sender` is not `tx.origin`.

There are 7 instances:

- *LandManager.sol* ( [100-100](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L100-L100), [113-113](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L113-L113), [128-128](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L128-L128), [170-170](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L170-L170), [196-196](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L196-L196), [224-224](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L224-L224), [225-225](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L225-L225) ):

```solidity
100:         emit TaxRateChanged(landlord, oldTaxRate, newTaxRate);

113:         emit UpdatePlotsMeta(mainAccount);

128:         emit UpdatePlotsMeta(landlord);

170:         emit FarmPlotTaken(toilerState[tokenId], tokenId);

196:         emit FarmPlotLeave(_toiler.landlord, tokenId, _toiler.plotId);

224:         emit FarmPlotLeave(_toiler.landlord, tokenId, oldPlotId);

225:         emit FarmPlotTaken(toilerState[tokenId], tokenId);
```

### [N-12] Imports could be organized more systematically

The contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files. The examples below do not follow this layout.

There is 1 instance:

- *LandManager.sol* ( [8-8](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L8-L8) ):

```solidity
/// @audit Out of order with the prev import️ ⬆
8: import "../interfaces/INFTAttributesManager.sol";
```

### [N-13] There is no need to initialize bool variables with `false`

Since the bool variables are automatically set to `false` when created, it is redundant to initialize it with `false` again.

There is 1 instance:

- *LandManager.sol* ( [317-317](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L317-L317) ):

```solidity
317:         bool found = false;
```

### [N-14] Named imports of parent contracts are missing

Consider adding named imports for all parent contracts.

There are 2 instances:

- *LandManager.sol* ( [11-11](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L11-L11) ):

```solidity
/// @audit BaseBlastManagerUpgradeable
/// @audit ILandManager
11: contract LandManager is BaseBlastManagerUpgradeable, ILandManager {
```

### [N-15] Constants should be put on the left side of comparisons

Putting constants on the left side of comparison statements is a best practice known as [Yoda conditions](https://en.wikipedia.org/wiki/Yoda_conditions).
Although solidity's static typing system prevents accidental assignments within conditionals, adopting this practice can improve code readability and consistency, especially when working across multiple languages.

There are 6 instances:

- *LandManager.sol* ( [96](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L96), [106](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L106), [119](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L119), [178](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L178), [207](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L207), [340](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L340) ):

```solidity
/// @audit put `0` on the left
96:         if (plotMetadata[landlord].lastUpdated == 0)

/// @audit put `0` on the left
106:         if (plotMetadata[mainAccount].lastUpdated != 0)

/// @audit put `0` on the left
119:         if (plotMetadata[landlord].lastUpdated == 0) {

/// @audit put `address(0)` on the left
178:         if (_toiler.landlord == address(0)) revert NotStakedError();

/// @audit put `address(0)` on the left
207:         if (_toiler.landlord == address(0)) revert NotStakedError();

/// @audit put `0` on the left
340:         if (_player.registrationDate == 0) revert PlayerNotRegisteredError();
```

### [N-16] High cyclomatic complexity

Consider breaking down these blocks into more manageable units, by splitting things into utility functions, by reducing nesting, and by using early returns.

<details>
<summary>There is 1 instance (click to show):</summary>

- *LandManager.sol* ( [131-171](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L131-L171) ):

```solidity
131:     function stakeMunchable(
132:         address landlord,
133:         uint256 tokenId,
134:         uint256 plotId
135:     ) external override forceFarmPlots(msg.sender) notPaused {
136:         (address mainAccount, ) = _getMainAccountRequireRegistered(msg.sender);
137:         if (landlord == mainAccount) revert CantStakeToSelfError();
138:         if (plotOccupied[landlord][plotId].occupied)
139:             revert OccupiedPlotError(landlord, plotId);
140:         if (munchablesStaked[mainAccount].length > 10)
141:             revert TooManyStakedMunchiesError();
142:         if (munchNFT.ownerOf(tokenId) != mainAccount)
143:             revert InvalidOwnerError();
144: 
145:         uint256 totalPlotsAvail = _getNumPlots(landlord);
146:         if (plotId >= totalPlotsAvail) revert PlotTooHighError();
147: 
148:         if (
149:             !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
150:             munchNFT.getApproved(tokenId) != address(this)
151:         ) revert NotApprovedError();
152:         munchNFT.transferFrom(mainAccount, address(this), tokenId);
153: 
154:         plotOccupied[landlord][plotId] = Plot({
155:             occupied: true,
156:             tokenId: tokenId
157:         });
158: 
159:         munchablesStaked[mainAccount].push(tokenId);
160:         munchableOwner[tokenId] = mainAccount;
161: 
162:         toilerState[tokenId] = ToilerState({
163:             lastToilDate: block.timestamp,
164:             plotId: plotId,
165:             landlord: landlord,
166:             latestTaxRate: plotMetadata[landlord].currentTaxRate,
167:             dirty: false
168:         });
169: 
170:         emit FarmPlotTaken(toilerState[tokenId], tokenId);
171:     }
```

</details>

### [N-17] Unnecessary casting

Unnecessary castings can be removed.

<details>
<summary>There are 9 instances (click to show):</summary>

- *LandManager.sol* ( [53-53](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L53-L53), [56-56](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L56-L56), [60-60](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L60-L60), [65-65](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L65-L65), [68-68](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L68-L68), [71-71](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L71-L71), [74-74](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L74-L74), [77-77](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L77-L77), [271-276](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L271-L276) ):

```solidity
/// @audit IConfigStorage(configStorage)
53:             IConfigStorage(configStorage).getAddress(StorageKey.LockManager)

/// @audit IConfigStorage(configStorage)
56:             IConfigStorage(configStorage).getAddress(StorageKey.AccountManager)

/// @audit IConfigStorage(configStorage)
60:             IConfigStorage(configStorage).getAddress(

/// @audit IConfigStorage(configStorage)
65:         MIN_TAX_RATE = IConfigStorage(configStorage).getUint(

/// @audit IConfigStorage(configStorage)
68:         MAX_TAX_RATE = IConfigStorage(configStorage).getUint(

/// @audit IConfigStorage(configStorage)
71:         DEFAULT_TAX_RATE = IConfigStorage(configStorage).getUint(

/// @audit IConfigStorage(configStorage)
74:         BASE_SCHNIBBLE_RATE = IConfigStorage(configStorage).getUint(

/// @audit IConfigStorage(configStorage)
77:         PRICE_PER_PLOT = IConfigStorage(configStorage).getUint(

/// @audit int16(
///                            REALM_BONUSES[
///                                (uint256(immutableAttributes.realm) * 5) +
///                                    uint256(landlordMetadata.snuggeryRealm)
///                            ]
///                        )
271:                 int16(
272:                     REALM_BONUSES[
273:                         (uint256(immutableAttributes.realm) * 5) +
274:                             uint256(landlordMetadata.snuggeryRealm)
275:                     ]
276:                 ) +
```

</details>

### [N-18] Consider using `delete` rather than assigning zero to clear values

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

There is 1 instance:

- *LandManager.sol* ( [192-192](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L192-L192) ):

```solidity
192:         munchableOwner[tokenId] = address(0);
```

### [N-19] Consider using named function arguments

When calling functions with multiple arguments, consider using [named function parameters](https://docs.soliditylang.org/en/latest/control-structures.html#function-calls-with-named-parameters), rather than positional ones.

There are 2 instances:

- *LandManager.sol* ( [152-152](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L152-L152), [195-195](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L195-L195) ):

```solidity
152:         munchNFT.transferFrom(mainAccount, address(this), tokenId);

195:         munchNFT.transferFrom(address(this), mainAccount, tokenId);
```

### [N-20] Named returns are recommended

Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

There are 2 instances:

- *LandManager.sol* ( [332-334](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L332-L334), [344-344](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L344-L344) ):

```solidity
332:     function _getMainAccountRequireRegistered(
333:         address _account
334:     ) internal view returns (address, MunchablesCommonLib.Player memory) {

344:     function _getNumPlots(address _account) internal view returns (uint256) {
```

### [N-21] Returning a struct instead of a bunch of variables is better

If a function returns [too many variables](https://docs.soliditylang.org/en/v0.8.21/contracts.html#returning-multiple-values), replacing them with a struct can improve code readability, maintainability and reusability.

There is 1 instance:

- *LandManager.sol* ( [332-334](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L332-L334) ):

```solidity
332:     function _getMainAccountRequireRegistered(
333:         address _account
334:     ) internal view returns (address, MunchablesCommonLib.Player memory) {
```

### [N-22] Contract variables should have comments

Consider adding some comments on non-public contract variables to explain what they are supposed to do. This will help for future code reviews.

There are 11 instances:

- *LandManager.sol* ( [12-12](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L12-L12), [13-13](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L13-L13), [14-14](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L14-L14), [15-15](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L15-L15), [16-16](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L16-L16), [17-17](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L17-L17), [18-18](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L18-L18), [31-31](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L31-L31), [32-32](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L32-L32), [33-33](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L33-L33), [34-34](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L34-L34) ):

```solidity
12:     uint256 MIN_TAX_RATE;

13:     uint256 MAX_TAX_RATE;

14:     uint256 DEFAULT_TAX_RATE;

15:     uint256 BASE_SCHNIBBLE_RATE;

16:     uint256 PRICE_PER_PLOT;

17:     int16[] REALM_BONUSES;

18:     uint8[] RARITY_BONUSES;

31:     ILockManager lockManager;

32:     IAccountManager accountManager;

33:     IERC721 munchNFT;

34:     INFTAttributesManager nftAttributesManager;
```

### [N-23] Multiple mappings with same keys can be combined into a single struct mapping for readability

Well-organized data structures make code reviews easier, which may lead to fewer bugs. Consider combining related mappings into mappings to structs, so it's clear what data is related.

There are 5 instances:

- *LandManager.sol* ( [21-21](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L21-L21), [23-23](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L23-L23), [25-25](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L25-L25), [27-27](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L27-L27), [29-29](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L29-L29) ):

```solidity
21:     mapping(address => PlotMetadata) plotMetadata;

23:     mapping(address => mapping(uint256 => Plot)) plotOccupied;

25:     mapping(uint256 => address) munchableOwner;

27:     mapping(address => uint256[]) munchablesStaked;

29:     mapping(uint256 => ToilerState) toilerState;
```

### [N-24] Missing event for critical changes

Events should be emitted when critical changes are made to the contracts.

There is 1 instance:

- *LandManager.sol* ( [312-330](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L312-L330) ):

```solidity
312:     function _removeTokenIdFromStakedList(
313:         address mainAccount,
314:         uint256 tokenId
315:     ) internal {
316:         uint256 stakedLength = munchablesStaked[mainAccount].length;
317:         bool found = false;
318:         for (uint256 i = 0; i < stakedLength; i++) {
319:             if (munchablesStaked[mainAccount][i] == tokenId) {
320:                 munchablesStaked[mainAccount][i] = munchablesStaked[
321:                     mainAccount
322:                 ][stakedLength - 1];
323:                 found = true;
324:                 munchablesStaked[mainAccount].pop();
325:                 break;
326:             }
327:         }
328: 
329:         if (!found) revert InvalidTokenIdError();
330:     }
```

### [N-25] Consider adding emergency-stop functionality

Adding a way to quickly halt protocol functionality in an emergency, rather than having to pause individual contracts one-by-one, will make in-progress hack mitigation faster and much less stressful.

There is 1 instance:

- *LandManager.sol* ( [11-11](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L11-L11) ):

```solidity
11: contract LandManager is BaseBlastManagerUpgradeable, ILandManager {
```

### [N-26] Missing checks for uint state variable assignments

Consider whether reasonable bounds checks for variables would be useful.

There are 5 instances:

- *LandManager.sol* ( [65-67](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L65-L67), [68-70](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L68-L70), [71-73](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L71-L73), [74-76](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L74-L76), [77-79](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L77-L79) ):

```solidity
65:         MIN_TAX_RATE = IConfigStorage(configStorage).getUint(
66:             StorageKey.LockManager
67:         );

68:         MAX_TAX_RATE = IConfigStorage(configStorage).getUint(
69:             StorageKey.AccountManager
70:         );

71:         DEFAULT_TAX_RATE = IConfigStorage(configStorage).getUint(
72:             StorageKey.ClaimManager
73:         );

74:         BASE_SCHNIBBLE_RATE = IConfigStorage(configStorage).getUint(
75:             StorageKey.MigrationManager
76:         );

77:         PRICE_PER_PLOT = IConfigStorage(configStorage).getUint(
78:             StorageKey.NFTOverlord
79:         );
```

### [N-27] Large or complicated code bases should implement invariant tests

This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts.
Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold.
Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.

There is 1 instance:

- Global finding

### [N-28] The default value is manually set when it is declared

In instances where a new variable is defined, there is no need to set it to it's default value.

There are 2 instances:

- *LandManager.sol* ( [247-247](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L247-L247), [318-318](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L318-L318) ):

```solidity
247:         for (uint8 i = 0; i < staked.length; i++) {

318:         for (uint256 i = 0; i < stakedLength; i++) {
```

### [N-29] Contracts should have all `public`/`external` functions exposed by `interface`s

All `external`/`public` functions should extend an `interface`. This is useful to ensure that the whole API is extracted and can be more easily integrated by other projects.

There is 1 instance:

- *LandManager.sol* ( [11](https://github.com/code-423n4/2024-07-munchables/blob/94cf468aaabf526b7a8319f7eba34014ccebe7b9/src/managers/LandManager.sol#L11) ):

```solidity
11: contract LandManager is BaseBlastManagerUpgradeable, ILandManager {
```
