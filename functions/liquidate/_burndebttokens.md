# \_burnDebtTokens

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

<img src="../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

Burn either stable or variable debt tokens, based on the amount of **`actualDebtToLiquidate`,** which was previously established.







## ERROR: Follow-up with Aave&#x20;

**\_calculateDebt** returns `vars.`**`userVariableDebt`** and `vars.`**`actualDebtToLiquidate`**

{% code fullWidth="true" %}
```solidity
// Assuming user has only variable debt, userTotalDebt = userVariableDebt
(uint256 userStableDebt, uint256 userVariableDebt) = Helpers.getUserCurrentDebt(params.user, debtReserveCache);
uint256 userTotalDebt = userStableDebt + userVariableDebt;

// Apply closeFactor
uint256 closeFactor = healthFactor > CLOSE_FACTOR_HF_THRESHOLD ? DEFAULT_LIQUIDATION_CLOSE_FACTOR : MAX_LIQUIDATION_CLOSE_FACTOR;
uint256 maxLiquidatableDebt = userTotalDebt.percentMul(closeFactor);

//Is Liquidator covering full or partial?
uint256 actualDebtToLiquidate = params.debtToCover > maxLiquidatableDebt ? maxLiquidatableDebt : params.debtToCover;
return (userVariableDebt, userTotalDebt, actualDebtToLiquidate);
```
{% endcode %}

From here it is established that&#x20;

* **userVariableDebt** > **maxLiquidatableDebt** (due to closeFactor)
* actualDebtToLiquidate =&#x20;
  * maxLiquidatableDebt, OR
  * params.debtToCover (when debtToCover < maxLiquidatableDebt )

Conclusively, userVariableDebt > actualDebtToLiquidate == maxLiquidatableDebt&#x20;

* if **actualDebtToLiquidate** == maxLiquidatableDebt**:**&#x20;
  * **userVariableDebt** > **actualDebtToLiquidate**&#x20;
* if **actualDebtToLiquidate** == debtToCover **:**&#x20;
  * **userVariableDebt** > **actualDebtToLiquidate**&#x20;

This means that userVariableDebt will never be < **actualDebtToLiquidate.** The if block based on condition `(vars.userVariableDebt != 0)` will never execute.
