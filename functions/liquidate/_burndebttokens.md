# 🚧 \_burnDebtTokens

## Overview

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [x] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [x] \_calculateDebt
* [x] validateLiquidationCall
* [x] getConfigurationData
* [x] calculateAvailableCollateralToLiquidate
* [x] <mark style="color:orange;">setBorrowing</mark>
* [x] <mark style="color:orange;">setUsingAsCollateral</mark>
* [ ] \_burnDebtTokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] <mark style="color:orange;">updateIsolatedDebtIfIsolated</mark>
* [ ] liquidate/burn collateral
* [ ] liquidation fee
* [ ] Wrap-up

## \_burnDebtTokens

<img src="../../.gitbook/assets/file.excalidraw (1) (2).svg" alt="" class="gitbook-drawing">

Burn either stable or variable debt tokens, based on the amount of **`actualDebtToLiquidate`,** which was previously established.

