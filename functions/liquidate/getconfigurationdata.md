# getConfigurationData

## Overview

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [x] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [x] \_calculateDebt
* [x] validateLiquidationCall
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

## getConfigurationData

<img src="../../.gitbook/assets/file.excalidraw (22).svg" alt="" class="gitbook-drawing">

* get liquidation bonus of collateral from its ReserveData struct: `liquidationBonus`
* get oracle addresses for both debt and collateral assets:&#x20;
  * `collateralPriceSource`, `debtPriceSource`
* **If user is in some E-mode category**
  * get `eModePriceSource` -> if defined, will overwrite `debtPriceSource` that was obtained earlier
  * **If user's e-mode and collateral e-mode categories match:**
    * overwrite `liquidationBonus` with the e-mode's liquidation bonus
    * if `eModePriceSource` is defined, overwrite `collateralPriceSource` that was obtained earlier
