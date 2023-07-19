# isFirstSupply

## Overview

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] validateSupply
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [x] transfer & mint
* [ ] If isFirstSupply

This is the last execution block for a supply action. This if block only executes in the event of a user's very first supply action for a specific asset.

If on minting ATokens, `isFirstSupply` is returned as `TRUE`, and the nested `validateUseAsCollateral` returns `TRUE` as well,&#x20;

* `isFirstSupply` : `TRUE` => user's first time supplying this specific asset
* `validateUseAsCollateral` : `TRUE` => either:
  * `isUsingAnyCollateralAny`: `TRUE` => user never supplied any asset ever,
  * OR,
  * `!isolationMode && getDebtCeiling == 0`: TRUE => there is no debt ceiling on incoming supplied asset AND the user is not in isolation mode.

<img src="../../../.gitbook/assets/file.excalidraw (1) (1) (1).svg" alt="" class="gitbook-drawing">

## isFirstSupply

* captures the return value from `mint`
* `TRUE` if user has no prior supply actions, and therefore a pre-dating zero supply balance

If `TRUE`, execution moves into the nested if block, with the return value of `validateUseAsCollateral` as the condition.

### validateUseAsCollateral

This function checks if the incoming asset can be used as collateral. If the user is previously in [isolated mode](../../../aave-features/risk-management/isolation-mode.md), other assets cannot be supplied as collateral at the same time.&#x20;

{% hint style="info" %}
Supplied assets are automatically treated as collateral, if possible.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

#### isUsingAsCollateralAny

* checks if user has been supplying ANY asset as collateral
* returns `FALSE` if user has NOT supplied ANY asset so far

{% hint style="info" %}
[`isUsingAsCollateralAny` breakdown](isusingascollateralone-isusingascollateralany.md)
{% endhint %}

If the user has never supplied any assets to date, (`isUsingAsCollateralAny` returns `FALSE`) `validateUseAsCollateral` will exit early and return `TRUE`.&#x20;

* Execution will then proceed to [`setUsingAsCollateral`](./#setusingascollateral); this will engage in first-time user configuration setup.

{% hint style="success" %}
User cannot be in isolation mode if he has no supplied action to date.&#x20;
{% endhint %}

However, if `isUsingAsCollateralAny` is true; user has executed supply action sometime before.  This means, we must check if the user is currently in **isolation mode;** achieved by `getIsolationMode`.

* If he is, it could cause conflict with the new supply action.
* Additionally, there would be no need to execute `setUsingAsCollateral`, as this is not a first-time user.

### `getIsolationMode`

* if user is supplying only one type of collateral, get the debt ceiling of the collateral to check if its an isolated asset.
  * returns `TRUE`, along with value of debt ceiling
* If the user is supplying multiple types of collateral, clearly he is not in isolation mode, returns `FALSE`.

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

#### Visual aid

<img src="../../../.gitbook/assets/file.excalidraw (11).svg" alt="" class="gitbook-drawing">

* isUsingCollateralOne: checks if the user has been supplying only 1 asset as collateral

#### return (!isolationMode && getDebtCeiling == 0)

`!isolationMode && getDebtCeiling == 0`:

* TRUE => there is no debt ceiling on incoming supplied asset AND the user is not in isolation mode.

## setUsingAsCollateral

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

#### `(reserveIndex << 1) + 1`

* This bitwise operation doubles the index and increments it by 1
* Creates the necessary offset to manipulate the relevant bits to the asset of specified index
* For more in-depth explanation see: [isUsingCollateral](../../withdraw/collateral-check.md#example-check-if-the-user-has-been-using-the-reserve-at-index-2-as-collateral)

#### **`bit = 1 << offset`**

<img src="../../../.gitbook/assets/file.excalidraw (13).svg" alt="" class="gitbook-drawing">

If `usingAsCollateral` is **true**, indicating that the reserve should be marked as used for collateral,&#x20;

* bitwise OR operation between `data` and the `bit` value (`self.data |= bit`).&#x20;
* sets the corresponding bit in the `self.data` variable to `1`

If `usingAsCollateral` is **false**, indicating that the reserve should NOT be marked as collateral

* bitwise AND operation between `data` and the `bit` value (`self.data &= bit`).&#x20;
* sets the corresponding bit in the `self.data` variable to `0`
