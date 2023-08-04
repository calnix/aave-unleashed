# liquidation Fee

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
* [x] liquidate/burn collateral
* [ ] liquidation fee
* [ ] Wrap-up

## Liquidation fee

Liquidation fee is taken from the user and transferred to treasury.

<figure><img src="../../.gitbook/assets/image (1) (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>AToken.sol</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>AToken.sol</p></figcaption></figure>

* ATokens are transferred to the treasury as liquidation fee.&#x20;
* validate is set to false, so no HF check is done.

