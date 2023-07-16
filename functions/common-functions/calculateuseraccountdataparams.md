# calculateUserAccountDataParams

This function calculates and returns the following user data across all the assets:

* `totalCollateralInBaseCurrency`
* `totalDebtInBaseCurrency`
* `avgLTV`
* `avgLiquidationThreshold`
* `healthFactor`
* `hasZeroLtvCollateral`

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### 4. Set decimals and oracle interface

* Define the decimal precision of 1 unit if the asset (`1 Ether = 10**18` | `1 USDC = 10**6`)
* Define the oracle interface

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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

### 5. If asset is used as collateral

If the asset's liquidation threshold is defined and it is being used by the user as collateral, execute the following.

* get user's balance in base CCY and increment `totalCollateralInBaseCurrency`
* `totalCollateralInBaseCurrency` will value the user's total collateral across all asset classes, normalized into the same base currency.&#x20;
* E.g. get user's total collateral in USD.



{% hint style="info" %}
Each market has an AaveOracle contract where you can query token prices in the base currency. BaseCCY:&#x20;

* ETH on V2 mainnet/polygon&#x20;
* USD on all other markets)
{% endhint %}
