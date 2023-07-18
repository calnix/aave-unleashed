# calculateAvailableCollateralToLiquidate

## Overview

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [x] <mark style="color:orange;">cache + updateState + get health factor</mark>
* [x] \_calculateDebt
* [x] validateLiquidationCall
* [x] getConfigurationData
* [ ] calculateAvailableCollateralToLiquidate
* [ ] <mark style="color:orange;">setBorrowing</mark>
* [ ] <mark style="color:orange;">setUsingAsCollateral</mark>
* [ ] \_burnDebtTokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] <mark style="color:orange;">updateIsolatedDebtIfIsolated</mark>
* [ ] liquidate/burn collateral
* [ ] liquidation fee
* [ ] Wrap-up

## \_calculateAvailableCollateralToLiquidate

Calculates how much of a specific collateral can be liquidated, given a certain amount of debt asset.

![](<../../.gitbook/assets/image (5).png>)

* `actualDebtToLiquidate` is passed as the value for parameter`debtToCover`
* `actualDebtToLiquidate` => user's debt \* close factor
* `actualDebtToLiquidate` was obtained in \_calculateDebt

Returns

* collateralAmount to be liquidated (baseCollateral + liq. bonus)
* debtAmountNeeded (baseCollateral valued in debt asset)

{% hint style="info" %}
Liquidator repays debt and take equivalent value in collateral from user. However, user must additionally pay a haircut to the liquidator as a penalty.&#x20;
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw (26).svg" alt="" class="gitbook-drawing">

### 1. Get prices of both collateral and debt assets&#x20;

<figure><img src="../../.gitbook/assets/image (12).png" alt="" width="529"><figcaption></figcaption></figure>

### 2. Define 1 unit of each asset

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Define what 1 unit of each asset is

* 1 unit of ETH => 1e18
* 1 unit of USDC => 1e6

Obtain decimals; see [getDecimals](../common-functions/getdecimals.md).

### 3. Get liquidation protocol fee

When liquidating, the protocol may apply a liquidation fee that goes to the protocol's treasury.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

* getLiquidationProtocolFee applies a bitmask and bitwise operations similar to getFlags or getDecimals.
* See Common functions section or the Bitmap function to understand.

### 4. Calculate amount of collateral to liquidate (+ liq. bonus)

By now we have established how much debt the liquidator will **repay**: `actualDebtToLiquidate`

We then need to value the debt in collateral terms. This is expressed as `baseCollateral`.

{% hint style="info" %}
Liquidator repays debt to protocol, and takes the equivalent value in the form of collateral asset, from the user. (ignoring liq. bonus)
{% endhint %}

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

**Let's reorder the operations so it is clearer:**

```
// value debt in USD
(debtToCover/debtAssetUnit * debtPrice) = debt[USD]

// how many units of collateral 
debt[USD] / collateralPrice = debt in units of collateral, collateralAmtUnits

// express units of collateral in full decimals
collateralAmtUnits * collateralAssetUnit = collateral "in wei"

// baseCollateral == collateral "in wei"
```

Please note the "in wei" is simply meant to indicate that the collateral is now expressed in the full precision of its native decimals; this could be 6 for USDC or 18 for Ether. "in wei" is subjective.

`baseCollateral` is the equivalent amount of collateral for the specified debt to be repaid. Now we need to apply the liquidation bonus on it.

```
// liqudiationBonus expressed as 1.XX
maxCollateralToLiquidate = baseCollateral * liquidationBonus
```

The `liquidationBonus` is expressed as a percentage **1.XX:**

* for Ether, it is `10500`
* Percentages in Aave are defined with 2 decimals of precision (100.00).
  * 1e4 => 100.00%
* Therefore, `10500` => `105.00` => `1.05`

Hence, instead of obtained the `liquidationBonus` and subsequently adding it to the `baseCollateral`, in two steps, we can do so in 1 step.&#x20;

{% hint style="info" %}
Borrowers on liquidation lose the equivalent amount of collateral to the debt that was repaid to make them healthy AND suffer a haircut on top of that amount, known as the liquidation penalty.&#x20;

Liquidation penalty is the liquidation bonus given as incentive to liquidators.&#x20;
{% endhint %}

### 5. Does the user have sufficient collateral?

With finding `maxCollateralToLiquidate` we have established how much collateral the user must give up in this process. Does he have enough?

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

If `maxCollateralToLiquidate > userCollateralBalance` :&#x20;

* user has insufficient collateral as per what the liquidator is willing to repay
* calculate the debt repay amount based on user's available collateral&#x20;
* this will be the new debt repay amount, overriding what the liquidator initially supplied&#x20;

### 6. Liquidation Protocol Fee

Earlier in step 3 we obtained the liquidation protocol fee. Now we apply it.

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

```solidity
// get baseCollateral
collateralAmt / liquidationBonus = (baseCollateral + liq. bonus) / 1.05 = baseCollateral 

//calculate liquidation bonus
bonusCollateral = collateralAmt - (baseCollateral)
bonusCollateral = liq. bonus

// apply fee
liquidationProtocolFee = bonusCollateral * liquidationProtocolFeePercentage 
```

The liquidation protocol fee is applied upon the liquidation bonus. Effectively, liquidators pay a small percentage of their bonus to the protocol's treasury.&#x20;

{% hint style="info" %}
The liquidation protocol fee is expressed as percentage. For Ether on mainnet, it is `1000`

* `1000` -> `10.00%`&#x20;
* `liquidators pay 10% of their bonus to treasury`
{% endhint %}



{% hint style="success" %}
**User**&#x20;

* pays liquidator liquidation bonus
* loses collateral equivalent to the debt that was repaid by liquidator
* **collateral lost = (repay + liq. bonus)**

**Liquidator**

* gains liquidation bonus from user
* pays liquidation protocol fee
* **profit = (liquidation bonus - protocol fee - gas)**
{% endhint %}
