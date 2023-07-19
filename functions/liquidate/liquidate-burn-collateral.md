# liquidate/burn collateral

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
* [x] \_burnDebtTokens
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [x] <mark style="color:orange;">updateIsolatedDebtIfIsolated</mark>
* [ ] liquidate/burn collateral
* [ ] liquidation fee
* [ ] Wrap-up

## liquidate/burn collateral

<img src="../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

If liquidator has opted to receive aTokens in payment (**`params.receiveAToken == true`**)

* `_liquidateATokens` will be executed
* **Else**: \_burnCollatearlATokens will be executed

## `_liquidateATokens`&#x20;

Transfers aTokens from the target user to the liquidator.&#x20;

* Amount transferred: debtRepaid + liquidation penalty  (in collateral terms)
* If the liquidator did not have aTokens of this kind previously,
  * validateUseAsCollateral
  * setUsingAsCollateral

validateUseAsCollateral ensures that liquidator is not in isolation mode and the incoming asset is not an isolation mode asset.&#x20;

If these conditions are met, the reserve transferred to the liquidator is set to be used as collateral.&#x20;

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

`transferOnLiquidiation` calls `_transfer`, which executes the transfer of aTokens.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* `validate` is set to **`false`**, therefore `finalizeTransfer` is not executed

For explanations on validateUseAsCollateral and setUsingAsCollateral, please see the supply section on the following segments:

* [validateUseAsCollateral](../supply/isfirstsupply/#validateuseascollateral)
* &#x20;[setUsingAsCollateral](../supply/isfirstsupply/#setusingascollateral)

{% hint style="info" %}
Where possible, Aave opts to set supplied assets as collateral automatically. The exception to this are isolated assets.&#x20;
{% endhint %}
