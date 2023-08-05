# getIsolationModeState

## Overview

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [ ] getIsolationMode
* [ ] validateBorrow
* [ ] mint debt token
* [ ] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

<img src="../../.gitbook/assets/file.excalidraw (6).svg" alt="" class="gitbook-drawing">

## getIsolationModeState

Check if user is in isolation mode, if so, return the isolated asset address and its corresponding debt ceiling.

{% hint style="info" %}
Isolation mode: user can only use the specific isolation asset as collateral
{% endhint %}

There 3 other functions within `getIsolationModeState` which we will cover:

1. `isUsingAsCollateralOne`
2. `_getFirstAssetIdByMask`
3. `getDebtCeiling`

Essentially,&#x20;

1. `isUsingAsCollateralOne`: check if user has ONLY ONE asset as collateral (any one).
2. If check returns true, possible that the user is in isolation mode.
   * get and return isolation details: `(true, assetAddress, debtCeiling)`
3. `_getFirstAssetIdByMask` is used to obtain the `assetID` of this asset.&#x20;
   * Since `isUsingAsCollateralOne` is `true`,&#x20;
   * only 1 asset to ID, no need to worry about multiple assets
4. If `isUsingAsCollateralOne` is `false`&#x20;
   * user has multiple reserves as collateral&#x20;
   * not in Isolation mode&#x20;
   * return `(false, address(0), 0)`

### `isUsingAsCollateralOne`

Check if user has ONLY ONE asset as collateral (any one).

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

This function was explained previously when covering the `supply` function: [isUsingCollateralOne](../supply/isfirstsupply/isusingascollateralone-isusingascollateralany.md#isusingcollateralone)

### `_getFirstAssetIdByMask`

Returns the address of the first asset flagged in the bitmap given the corresponding bitmask.

* if collateral\_mask, find the first asset which has its collateral bit set to `1`

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

```solidity
uint256 internal constant COLLATERAL_MASK = 
0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA;
```

* `COLLATERAL_MASK` is passed as `mask`
* `bitmapData` is the binary string comprising of isolated collateral bits as explained in: [isUsingAsCollateralAny](../supply/isfirstsupply/isusingascollateralone-isusingascollateralany.md#isusingascollateralany)&#x20;

<img src="../../.gitbook/assets/file.excalidraw (8) (1).svg" alt="" class="gitbook-drawing">

As we can see, we can end with id of value 1 -> index of the first asset which has its collateral bit set to `1` is 1.&#x20;

### `getDebtCeiling`

With the previously obtained asset index, we can retrieve the `assetAddress` by the `reserveList` mapping. With the address, we can retrieve its corresponding `reservesData` struct, using it to obtain the debt ceiling.

```solidity
address assetAddress = reserveList[assetId];  //declared on PoolStorage.sol
uint256 ceiling = reservesData[assetAddress].configuration.getDebtCeiling();
```

<figure><img src="../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

`getDebtCeiling` will extract the debt ceiling value held within bits 212-251 in the `ReserveConfigurationMap`.

<details>

<summary>Reference: ReserveConfigurationMap</summary>

![](<../../.gitbook/assets/image (229) (1).png>)

</details>

`getDebtCeiling` utilizes the same approach as explained in [getReserveFactor](../../primer/bitmap-and-masks/#getreservefactor); see that section for an in-depth explanation.

{% hint style="info" %}
The debt ceiling value has decimals as dictated in **`ReserveConfiguration::DEBT_CEILING_DECIMALS`**, like so:

```solidity
uint256 public constant DEBT_CEILING_DECIMALS = 2;
```
{% endhint %}

