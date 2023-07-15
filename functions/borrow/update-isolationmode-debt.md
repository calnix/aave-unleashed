# update IsolationMode debt

## Overview

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] getIsolationMode
* [x] validateBorrow
* [x] mint debt token
* [x] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

<img src="../../.gitbook/assets/file.excalidraw (19).svg" alt="" class="gitbook-drawing">

The bool `isolationModeAction` was retrieved earlier, from `getIsolationModeState`.&#x20;

If `isolationModeAction` is `true`, this implies that the incoming borrow action is an isolated borrow as well.&#x20;

* get current `isolationModeTotalDebt` (from `reservesData`)
* increment it by the incoming stable debt amount
* assign the updated `isolationModeTotalDebt` to&#x20;
  * `reservesData[...].isolationModeTotalDebt`
  * nextIsolationModeTotalDebt

<figure><img src="../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

comment on decimals
