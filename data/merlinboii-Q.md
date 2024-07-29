# QA Report

## Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01) | Lack of getter functions | Low |
| [L-02](#l-02) | Lack of event emission for caller traceability | Low |
| [L-03](#l-03) | Misleading storage keys for rate configuration | Low |
| [L-04](#l-04) | Inconsistent pause state across protocol features | Low |
| [N-01](#n-01) | Redundant Approval Check in `stakeMunchable()` | Non-Critical |
</br>


## **[L-01] Lack of getter functions**<a name="l-01"></a>

Many variables in the `LandManager` contract are not publicly accessible, making it inconvenient for users to retrieve necessary information for making interaction decisions.

Missing Getter Functions:

* Availability of plots for each landlord.
* `toilerState` data.
* `munchablesStaked` information.
* `plotMetadata` details.

### Recommendation
Implement getter functions for these variables to allow users to easily access and retrieve the necessary data.
</br>

---

## **[L-02] Lack of event emission for caller traceability**<a name="l-02"></a>

The account mechanism allows sub-accounts to interact on behalf of the main account. There is a need to improve traceability by emitting events that indicate which account initiated actions.

The affected events:
* `event UpdatePlotsMeta`
* `event TaxRateChanged`
* `event FarmPlotTaken`
* `event FarmPlotLeave`
* `event FarmedSchnibbles`

### Recommendation
Implement event emissions that capture and log the caller information whenever an action is initiated. 
</br>

---

## **[L-03] Misleading storage keys for rate configuration**<a name="l-03"></a>

In the `LandManager` contract, tax rates such as `MIN_TAX_RATE`, `MAX_TAX_RATE`, and `DEFAULT_TAX_RATE`, as well as `BASE_SCHNIBBLE_RATE` and `PRICE_PER_PLOT`, are fetched from `ConfigStorage` using storage keys like `StorageKey.LockManager`, `StorageKey.AccountManager`, and `StorageKey.ClaimManager`.

These storage keys are not intuitively named for setting rates, which could lead to confusion and errors in configuration management.

### Recommendation
Rename the storage keys in `ConfigStorage` used for setting rates to more descriptive names that reflect their purpose.

</br>

---

## **[L-04] Inconsistent pause state across protocol features**<a name="l-04"></a>

The `pause` state should be consistent across the entire protocol system, as all features share the same configuration (`ConfigStorage`). 

Any changes to the configuration should require reconfiguration of all features to ensure consistency.

### Recommendation
Ensure that the `pause` state is consistently applied across all features within the protocol system.
</br>

---

## **[N-01] Redundant Approval Check in `stakeMunchable()`**<a name="n-01"></a>

In the `stakeMunchable()`, the approval check before calling the MunchNFT `transferFrom()` is unnecessary. The MunchNFT (ERC721) standard already implements approval checks before performing the actual transfer.

```solidity
File: src/managers/LandManager.sol

148:         if (
149:             !munchNFT.isApprovedForAll(mainAccount, address(this)) &&
150:             munchNFT.getApproved(tokenId) != address(this)
151:         ) revert NotApprovedError();

```

### Recommendation
Consider removing the redundant approval check before calling `transferFrom()` in the `stakeMunchable()`.

</br>

---