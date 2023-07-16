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

### 1. Check if asset isUsingAsCollateralOrBorrowing

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

If the asset is not being used as either, increment the counter and `continue`; skip the remaining block of code and progress to the next iteration.&#x20;
