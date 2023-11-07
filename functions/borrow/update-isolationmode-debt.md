# update IsolationMode debt

## Overview

<figure><img src="../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] getIsolationMode
* [x] validateBorrow
* [x] mint debt token
* [x] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

<img src="../../.gitbook/assets/file.excalidraw (4).svg" alt="" class="gitbook-drawing">

The bool `isolationModeAction` was retrieved earlier, from `getIsolationModeState`.&#x20;

If `isolationModeAction` is **`true`**, this implies that the incoming borrow action is an isolated borrow as well.&#x20;

* get current `isolationModeTotalDebt` (from `reservesData` mapping)
* increment it by the incoming stable debt amount
* assign the updated `isolationModeTotalDebt` to both:
  * `reservesData[...].isolationModeTotalDebt`
  * nextIsolationModeTotalDebt

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

**On decimals:**

The amount is divided by $$10^{reserveDecimals}$$to standardise amount into the number of units of assets.

**I.e.**: amount (in wei)/  $$10^{reserveDecimals}$$ = amount (in units of asset)

{% hint style="info" %}
* 1 USDC = 1e6
* 1 Ether = 1e18
{% endhint %}
