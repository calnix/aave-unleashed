# transfer underlying to user

## Overview

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] getIsolationMode
* [x] validateBorrow
* [x] mint debt token
* [x] setBorrowing&#x20;
* [x] update IsolationMode debt
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

{% hint style="info" %}
<mark style="color:orange;">.updateInterestRates</mark> is skipped as it has been explained well in prior sections. Please see: .[updateInterestRates](../common-functions/.updateinterestrates.md)
{% endhint %}

## transfer underlying to user

User initiated a borrow for a specific asset, in this step we will transfer this asset to the user.

<img src="../../.gitbook/assets/file.excalidraw (3).svg" alt="" class="gitbook-drawing">

* Asset is transferred to the user via safeTransfer.

{% hint style="info" %}
**`releaseUnderlying`** set to false in executeFlashLoan
{% endhint %}
