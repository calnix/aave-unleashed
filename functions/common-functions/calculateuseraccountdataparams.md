# calculateUserAccountDataParams

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

This function calculates and returns the following user data across all the assets:

* `totalCollateralInBaseCurrency`
* `totalDebtInBaseCurrency`
* `avgLTV`
* `avgLiquidationThreshold`
* `healthFactor`
* `hasZeroLtvCollateral`

## Visual Aid

<img src="../../.gitbook/assets/file.excalidraw (14).svg" alt="" class="gitbook-drawing">

## check if `userConfig` is empty

```solidity
    if (params.userConfig.isEmpty()) {
      return (0, 0, 0, 0, type(uint256).max, false);
    }
```

<figure><img src="../../.gitbook/assets/image (144).png" alt="" width="563"><figcaption><p>UserConfiguration.sol</p></figcaption></figure>

If all the bits in UserConfiguration is set to 0, `data == 0` will evaluate to be true. This indicates that the user did not undertake any borrow or supply as collateral action.

* returns health factor as `type(uint256).max`
* returns `hasZeroLTVCollateral` as **`false`**

## **If user is in e-mode**

We need to obtain e-mode specific info such as LTV, liquidationThreshold and eModeAssetPrice.

{% code overflow="wrap" %}
```solidity
if (params.userEModeCategory != 0) {
  (vars.eModeLtv, vars.eModeLiqThreshold, vars.eModeAssetPrice) = EModeLogic
    .getEModeConfiguration(
      eModeCategories[params.userEModeCategory],
      IPriceOracleGetter(params.oracle)
    );
}
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Get e-mode configuration details, by passing the user's eModeCategory id (`params.userEModeCategory`) into the mapping `eModeCategories.` This will return the `EModeCategory` struct.

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

`params.userEModeCategory` was obtained previously by passing `_usersEModeCategory[msg.sender]`

<figure><img src="../../.gitbook/assets/image (131).png" alt=""><figcaption><p>PoolStorage.sol</p></figcaption></figure>

The if statement exists purely to check if a `priceSource` was defined in `EModeCategory`. If no `priceSource` was defined, `params.oracle` is returned as the oracle address.

Else `params.oracle`, is overwritten with the `category.priceSource`.

## While loop

```solidity
while (vars.i < params.reservesCount) {...}
```

Loops through all the active reserves there are in the protocol. `reservesCount` is defined on PoolStorage.sol

{% code title="PoolStorage.sol" %}
```solidity
 // Maximum number of active reserves there have been in the protocol. It is the upper bound of the reserves list
  uint16 internal _reservesCount;
```
{% endcode %}

For each asset, the following is executed.&#x20;

### 1. Check if asset isUsingAsCollateralOrBorrowing

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

If the asset is not being used as either, increment the counter and `continue`; skip the remaining block of code and moving to the next `reserveIndex`.&#x20;

#### isUsingAsCollateralOrBorrowing

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* require statement performs a boundary check to ensure that `reserveIndex` value is within the valid range of `[0 - 127]`.
* If you are unclear on the bitmap manipulations, please see that section.

### 2. Check if zero address

```solidity
// reservesList: List of reserves as a map (reserveId => reserve).
        vars.currentReserveAddress = reservesList[vars.i];

    if (vars.currentReserveAddress == address(0)) {
      unchecked {
        ++vars.i;
      }
      continue;
    }
```

Get the asset address of the current iteration by passing the counter into mapping `reservesList`

* reservesList is defined on PoolStorage

If asset address is undefined, increment counter and continue.

{% hint style="warning" %}
Would there be gas savings by checking for zero address first, then followed by isUsingAsCollateralOrBorrow?&#x20;
{% endhint %}

### 3. Get asset's params

Now that we have established that the asset is defined and being used by the user as either collateral or borrowing, let us obtain the following key details:

* LTV
* liquidationThreshold
* decimals
* Emode category

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

This is achieved via `getParams`, which utilizes bitmasks to extract the relevant information from the ReserveConfigurationMap; which is a bitmap.

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

### 4. Set decimals and oracle interface

* Define the decimal precision of 1 unit if the asset (`1 Ether = 10**18` | `1 USDC = 10**6`)
* Define the oracle interface

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

If both the user and the asset are in the same e-mode category, and `vars.eModeAssetPrice !=0`, use `vars.eModeAssetPrice`

{% code overflow="wrap" %}
```solidity
// this was set earlier via getEModeConfiguration
vars.eModeAssetPrice = IPriceOracleGetter(params.oracle).getAssetPrice(eModePriceSource)
```
{% endcode %}

Else, default to using the following as the oracle interface:

```solidity
IPriceOracleGetter(params.oracle).getAssetPrice(vars.currentReserveAddress);
```

{% hint style="info" %}
Setting of oracles is crucial because we will be normalizing all of the user's collateral and debt to a common base currency; likely USD. This will allow us to calculate wallet-level metrics like LTV and liquidation threshold and consequently the user's health factor.&#x20;
{% endhint %}

### 5. If asset is used as collateral

If the asset's liquidation threshold is defined and it is being used by the user as collateral, execute the following.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* get user's balance in base CCY and increment **`totalCollateralInBaseCurrency`**
* `totalCollateralInBaseCurrency` will be the sum of collateral across all asset classes, normalized into the base currency.&#x20;
* E.g. get user's total collateral in USD.

{% hint style="info" %}
Each market has an AaveOracle contract where you can query token prices in the base currency. BaseCCY:&#x20;

* ETH on V2 mainnet/polygon&#x20;
* USD on all other markets)
{% endhint %}

```solidity
// if user and asset in same emode category, returns true
vars.isInEModeCategory = EModeLogic.isInEModeCategory(params.userEModeCategory, vars.eModeAssetCategory);
```

**If the asset's LTV is defined: avgLTV**

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* calculate the user's max debt for each asset&#x20;
* the sum of these across all assets will give us the numerator for the avgLTV calculation
* we will divide by `totalCollateralInBaseCurrency` at the end

$$
avgLTV = \frac{ \sum{Collateral_i \: in \: baseCCY \: \times \: LTV ratio _i}}{totalCollateralInBaseCurrency}
$$

{% hint style="info" %}
Loan to Value (”LTV”) ratio defines the maximum amount of assets that can be borrowed with a specific collateral.
{% endhint %}

**avgLiquidationThreshold**

For each wallet, the Liquidation Threshold is calculate as the weighted average of the Liquidation Thresholds of the collateral assets and their value:

$$Liquidation \: Threshold= \frac{ \sum{Collateral_i \: in \: baseCCY \: \times \: Liquidation \: Threshold_i}}{Total \: Collateral \: in \: baseCCY\:}$$

{% code overflow="wrap" %}
```solidity
vars.avgLiquidationThreshold += vars.userBalanceInBaseCurrency * (vars.isInEModeCategory ? vars.eModeLiqThreshold : vars.liquidationThreshold);
```
{% endcode %}

At this stage we simply look to obtain the numerator for the **avgLiquidationThreshold** calculation**.** Like avgLTV, the division will be done at the end, once the loop has been completed.&#x20;

{% hint style="info" %}
Liquidation threshold is the percentage at which a position is defined as **undercollateralised**. For example, a Liquidation threshold of 80% means that if the loan value rises above 80% of the collateral, the position is undercollateralised and could be liquidated.
{% endhint %}

### 6. isBorrowing

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

If user is borrowing this asset, calculate its value in base CCY, and increment `vars.totalDebtInBaseCurrency`

## Exit loop

Now that we have traversed across the entire universe of assets and increments the various necessary measures like&#x20;

* `totalCollateralInBaseCurrency`&#x20;
* `totalDebtInBaseCurrency`
* LTV, Liquidation threshold

We have the prerequisites to calculate a wallet's health factor.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

First we obtain **avgLtv** and **avgLiquidationThreshold** by dividing them each against `totalCollateralInBaseCurrency`.&#x20;

{% hint style="info" %}
Remember, we previously only obtained their respective numerators for the weighted calculation, in the while loop.&#x20;
{% endhint %}

Then the calculation for health factor:

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt="" width="451"><figcaption></figcaption></figure>

**`avgLiquidationThreshold`** was obtained by dividing the weighted sum by totalCollateral, therefore this can be expressed as:

$$
hf = \frac{ \sum{Collateral_{i, baseCCY} \: \times \: Liquidation \: Threshold_i} }{TotalDebt_{baseCCY}}
$$

* collateral \* liquidationThreshold => max debt possible for that collateral asset
* the ratio of the sum of max debt possible against the totalDebt presently held constitutes the health factor of a wallet&#x20;

#### **If hf < 1,**&#x20;

* numerator **<** denominator
* $$\sum{Collateral_{i, baseCCY} \: \times \: Liquidation \: Threshold_i}$$ **<**  $${TotalDebt_{baseCCY}}$$

**User's current total debt exceeds his max loan value possible; hence considered undercollateralized.**&#x20;
