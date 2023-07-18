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

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<img src="../../.gitbook/assets/file.excalidraw (29).svg" alt="" class="gitbook-drawing">

A valid liquidationCall must fulfil the following criteria:

* Status flags for both collateral and borrow asset must be ACTIVE and NOT PAUSED
* L2 check
* Ensure user's health factor < 1
* Collateral check
  * check that liquidation threshold is non-zero
  * confirm that user is using the collateral&#x20;

