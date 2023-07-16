# collateral check

## Overview

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] get user's AToken balance + amount to withdraw
* [x] validateWithdraw: confirm user has sufficient balance and asset isActive
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [ ] collateral check
* [ ] burn ATokens
* [ ] Ensure existing loans are not impacted&#x20;

{% hint style="info" %}
Interest rates are updated to account for the withdrawal of deposits, and therefore the reduction of supply. See [.updateInterestRates](../common-functions/.updateinterestrates.md)
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw.svg" alt="" class="gitbook-drawing">

## isUsingCollateral

* Transaction reverts on invalid `reserveIndex`

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Let's examine the bitwise manipulations with an example. In this example, we assume six bits for simplicity, w.r.t to `self.data`

#### **Example: Check if the user has been using the reserve at index 2 as collateral:**

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

It the result of the bitwise operations is not equal to 0, the function would return `true`. This indicates that the user has been using the reserve at `reserveIndex = 2` as collateral.

## setUsingAsCollateral

Set the collateral bit for a specific asset to either `0` or `1`, depending on bool `usingAsCollateral`.

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

#### `(reserveIndex << 1) + 1`

* This bitwise operation doubles the index and increments it by 1
* Creates the necessary offset to manipulate the relevant bits to the asset of specified index
* For more in-depth explanation see: [isUsingCollateral](collateral-check.md#example-check-if-the-user-has-been-using-the-reserve-at-index-2-as-collateral)

#### **`bit = 1 << offset`**

<img src="../../.gitbook/assets/file.excalidraw (13).svg" alt="" class="gitbook-drawing">

If `usingAsCollateral` is **true**, indicating that the reserve should be marked as used for collateral,&#x20;

* bitwise OR operation between `data` and the `bit` value (`self.data |= bit`).&#x20;
* sets the corresponding bit in the `self.data` variable to `1`

If `usingAsCollateral` is **false**, indicating that the reserve should NOT be marked as collateral

* bitwise AND operation between `data` and the `bit` value (`self.data &= bit`).&#x20;
* sets the corresponding bit in the `self.data` variable to `0`

