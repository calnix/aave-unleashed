# validateWithdraw

## Overview

<figure><img src="../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] get user's AToken balance + amount to withdraw
* [ ] validateWithdraw: confirm user has sufficient balance and asset isActive
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] collateral check
* [ ] burn ATokens
* [ ] Ensure existing loans are collateralized

## validateWithdraw

This function confirms user has sufficient balance meeting withdrawal requirements, and ensures the status of asset:

* isActive
* notPaused

<figure><img src="../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

* ensures that withdrawal amount is: $$0 < amount \leq userBalance$$, else revert
* checks status flags of asset

{% hint style="info" %}
To better understand getFlags, see: [getFlags](../common-functions/getflags/)
{% endhint %}
