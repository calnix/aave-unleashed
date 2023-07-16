# Ensure existing loans are not impacted

## Overview

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] get user's AToken balance + amount to withdraw
* [x] validateWithdraw: confirm user has sufficient balance and asset isActive
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [x] collateral check
* [x] burn ATokens
* [ ] Ensure existing loans are not impacted&#x20;

<img src="../../.gitbook/assets/file.excalidraw (12).svg" alt="" class="gitbook-drawing">

### isBorrowingAny

This function checks if a user has been borrowing **any** asset.

* Takes `UserConfigurationMap` as input, which contains a bitmap of the user's collaterals and borrows.

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Uses `BORROWING_MASK`, which is a bitmask that isolates the borrowing bits within `data`. `BORROWING_MASK` has a specific pattern of bits, with **alternating `1` and `0`**.

{% code overflow="wrap" fullWidth="false" %}
```solidity
// UserConfiguration.sol
uint256 internal constant BORROWING_MASK = 
0x5555555555555555555555555555555555555555555555555555555555555555;

// 0x55 -> Binary(1010101)
```
{% endcode %}

The function applies this mask to `data` using a bitwise AND operation (&). This effectively keeps only the bits in `data` that correspond to borrowing positions, while setting all other bits to 0.

If the result of the bitwise AND operation between `self.data` and `BORROWING_MASK` is not equal to `0`, it means that at least one bit in the borrowing position is set to `1`. This indicates that the user has been borrowing from at least one reserve.

<details>

<summary>Illustrated Explanation</summary>

To understand how the BORROWING\_MASK works as a bitmask, let's take a closer look at its binary representation:

**BORROWING\_MASK**: `0x5555555555555555555555555555555555555555555555555555555555555555`

In binary: `0101010101010101010101010101010101010101010101010101010101010101`

Each '0' represents a bit that will be set to 0 when the BORROWING\_MASK is applied, and each '1' represents a bit that will be preserved. When the BORROWING\_MASK is applied to the data field using a bitwise AND operation (&), the resulting value will have only the borrowing-related bits preserved, while all other bits will be set to 0.

For example, let's assume the data field contains the following binary representation:

**data**:\
`1111111111111111111111111111111111111111111111111111111111111111`

Applying the `BORROWING_MASK` using a bitwise AND operation:

**`data & BORROWING_MASK`:**

`1111111111111111111111111111111111111111111111111111111111111111 & 0101010101010101010101010101010101010101010101010101010101010101`

\=> **`0101010101010101010101010101010101010101010101010101010101010101`**

The resulting value preserves only the bits in the borrowing positions, while setting all other bits to 0. In this case, the preserved bits represent the borrowing positions, indicating which assets are borrowed by the user.

</details>

To summarize, `BORROWING_MASK` is a bitmask that preserves only the **borrowing-related bits** within the data field. Applying the BORROWING\_MASK using a bitwise AND operation helps identify the borrowed assets by setting all non-borrowing bits to 0 and preserving the borrowing bits.

If the user is borrowing any asset, AND has been using the asset about to be withdrawn as collateral => need to validate HF and LTV.

## Check loans (validate HF and LTV)

* calls on `validateHealthFactor` to return the bool `hasZeroLTVCollateral`

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

Etiher one of the following two conditions must be true for this function to not revert:

1. **`!hasZeroLtvCollateral`**: user has a non-zero LTV value across all assets
2. Asset being withdrawn has an LTV of `0`

Condition 1 serves as a sanity check, given that `validateHFAndLtv` is executed in the context that the user `isBorrowingAny` .

### validateHealthFactor

This function is primarily a wrapper around `.calculateUserAccountDataParams`.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

The require statement ensures that the calculated health factor (after withdrawal) is above the liquidation threshold. If its not, user will be unable to withdraw asset.

## `.calculateUserAccountDataParams`

This function calculates and returns the following user data across all the assets:

* `totalCollateralInBaseCurrency`
* `totalDebtInBaseCurrency`
* `avgLTV`
* `avgLiquidationThreshold`
* `healthFactor`
* `hasZeroLtvCollateral`

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

### check if `userConfig` is empty

```solidity
    if (params.userConfig.isEmpty()) {
      return (0, 0, 0, 0, type(uint256).max, false);
    }
```

<figure><img src="../../.gitbook/assets/image (144).png" alt="" width="563"><figcaption><p>UserConfiguration.sol</p></figcaption></figure>

If all the bits in UserConfiguration is set to 0, `data == 0` will evaluate to be true. This indicates that the user did not undertake any borrow or supply as collateral action.

* returns health factor as `type(uint256).max`
* returns `hasZeroLTVCollateral` as **`false`**

### **If user is in e-mode**

We need to obtain e-mode specific info such as LTV, liquidationThreshold and eModeAssetPrice.

{% code overflow="wrap" %}
```solidity
if (params.userEModeCategory != 0) {
  (vars.eModeLtv, vars.eModeLiqThreshold, vars.eModeAssetPrice) = EModeLogic
    .getEModeConfiguration(
      eModeCategories[params.userEModeCategory],
      IPriceOracleGetter(params.oracle)
    );
}
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Get e-mode configuration details, by passing the user's eModeCategory id (`params.userEModeCategory`) into the mapping `eModeCategories.`&#x20;

<figure><img src="../../.gitbook/assets/image (131).png" alt=""><figcaption><p>PoolStorage.sol</p></figcaption></figure>

`This will return the e`

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

