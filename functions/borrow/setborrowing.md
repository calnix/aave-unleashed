# setBorrowing

## Overview

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] getIsolationMode
* [x] validateBorrow
* [x] mint debt token
* [ ] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

<img src="../../.gitbook/assets/file.excalidraw (32).svg" alt="" class="gitbook-drawing">

**`bool ifFirstBorrowing`** captures the return value from either of the debt token's mint function. It is `true`, if the user had no debt previously, thereby making this incoming borrow action the user's very first one.

## setBorrowing

The function performs bitwise operations to set or unset the borrowing status for the specified reserve.

<figure><img src="../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

It calculates the bit position based on the `reserveIndex` and creates a bit mask by left shifting `1` by twice the `reserveIndex` value. This mask is captured by the bit variable.

**Setting Borrowing Status:** If `borrowing` is true, the function uses a bitwise OR operation (`|=`) to set the corresponding bit in the `self.data` bitmap. This indicates that the user is borrowing the specified reserve.

**Unsetting Borrowing Status:** If borrowing is false, the function uses a bitwise AND operation (`&=`) with the bitwise negation (`~bit`) to unset the corresponding bit in the `self.data` map. This indicates that the user is not borrowing the specified reserve.

{% hint style="info" %}
For in-depth illustration on how this function works, see: [setUsingAsCollateral](../supply/isfirstsupply/#setusingascollateral).

The process for both are similar, one sets the borrow flag, the other sets the collateral flag.
{% endhint %}
