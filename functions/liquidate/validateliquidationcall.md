# validateLiquidationCall

## Overview

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [x] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [x] \_calculateDebt
* [ ] validateLiquidationCall
* [ ] getConfigurationData
* [ ] calculateAvailableCollateralToLiquidate
* [ ] <mark style="color:orange;">setBorrowing</mark>
* [ ] <mark style="color:orange;">setUsingAsCollateral</mark>
* [ ] \_burnDebtTokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] <mark style="color:orange;">updateIsolatedDebtIfIsolated</mark>
* [ ] liquidate/burn collateral
* [ ] liquidation fee
* [ ] Wrap-up

## validateLiquidationCall

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<img src="../../.gitbook/assets/file.excalidraw (29).svg" alt="" class="gitbook-drawing">

A valid liquidationCall must fulfil the following criteria:

* Status flags for both collateral and borrow asset must be ACTIVE and NOT PAUSED
* L2 check
* Ensure user's health factor < 1
* Collateral check
  * check that liquidation threshold is non-zero
  * confirm that user is using the collateral: `isUsingAsCollateral`

Since the liquidator can claim a single collateral of choice - we need to check the collateral address passed to ensure that it is indeed a valid selection. Else we might liquidate an asset that was not marked as collateral.

{% hint style="info" %}
**Liquidation threshold**&#x20;

* is the percentage at which a loan is defined as under-collateralized.
* usually no more than 10-15% above LTV.

For example, a liquidation threshold of 80% means that if the loan value rises above 80% of the collateral, the loan could be liquidated.

If the loan reaches the liquidation threshold, Aave prevents a user from borrowing and the user must either partially close their position or provide more collateral.
{% endhint %}
