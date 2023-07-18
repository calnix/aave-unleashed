# \_calculateDebt

## Overview

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [x] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [ ] \_calculateDebt
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

## &#x20;\_calculateDebt

Calculates the total debt of the user and the actual amount to liquidate depending on the health factor. Returns:

* userVariableDebt
* userTotalDebt
* actualDebtToLiquidate

<img src="../../.gitbook/assets/file.excalidraw (25).svg" alt="" class="gitbook-drawing">

* [getUserCurrentDebt](../repay/get-current-debt.md) will return the user's balance of debt tokens (stable & variable)
* `healthFactor` was obtained previously through `calculateUserAccountData`
* If user's `healthFactor` > 0.95: liquidation close factor is 0.5 (50%.00)
* If user's `healthFactor` <= 0.95: liquidation close factor is 1.0 (100.00%)

{% hint style="info" %}
The close factor determines the maximum liquidatable debt, per valid liquidationCall().&#x20;

close factor = 0.5, means only a maximum of 50% of the debt can be liquidated within that liquidationCall.
{% endhint %}

**On Partial and Full liquidation**

If the liquidator's `debtToCover` is > `maxLiquidatableDebt`, set the `actualDebtToLiquidate` to be `maxLiquidatableDebt`.

Else, If the liquidator's `debtToCover` is **<=** `maxLiquidatableDebt`, set the `actualDebtToLiquidate` to be `debtToCover`.

