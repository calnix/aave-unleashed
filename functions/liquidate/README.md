---
description: liquidationCall()
---

# liquidate

## Overview

This function is called by liquidators to liquidate "bad loans". Accounts with a health factor of less than 1 are eligible for liquidation.

**Assuming user's healthFactor < 1:**

* if healthFactor > 0.95 => close factor set to 50%
* if healthFactor <= 0.95 => close factor set to 100%&#x20;

Liquidators are rewarded as they receive \_\_\_\_\_\_\_\_\_\_ of the collateral. Can opt to be paid in ATokens as well.&#x20;

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

<img src="../../.gitbook/assets/file.excalidraw (23).svg" alt="" class="gitbook-drawing">

* `executeLiquidationCall` contains the core logics&#x20;

## executeLiquidationCall

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [ ] \_caluclateDebt
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

We will breakdown and examine the unique sections of logics within **executeLiquidationCall**. Code delineated in orange are common functions and can be explored in that [section](../common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/CBWXwogzvmzFhZw3BcJs?darkMode=true" %}
