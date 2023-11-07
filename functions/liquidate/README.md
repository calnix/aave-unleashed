---
description: liquidationCall()
---

# liquidate

## Overview

This function is called by liquidators to liquidate "bad loans". Accounts with a health factor of less than 1 are eligible for liquidation.

**Assuming user's healthFactor < 1:**

* if healthFactor > 0.95 => close factor set to 50%
* if healthFactor <= 0.95 => close factor set to 100%&#x20;

Liquidators are awarded with liquidation bonus, which can be paid in ATokens or the underlying asset as well.&#x20;

{% hint style="info" %}
* user’s health factor: $$0.95 < hf < 1$$, the loan is eligible for a liquidation of 50%.
* user’s health factor: $$hf <= 0.95$$, the loan is eligible for a liquidation of 100%.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

<img src="../../.gitbook/assets/file.excalidraw (35).svg" alt="" class="gitbook-drawing">

* `executeLiquidationCall` contains the core logics&#x20;

## executeLiquidationCall

<figure><img src="../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

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
