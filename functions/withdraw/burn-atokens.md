# burn ATokens

## Overview

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] get user's AToken balance + amount to withdraw
* [x] validateWithdraw: confirm user has sufficient balance and asset isActive
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [x] collateral check
* [ ] burn ATokens
* [ ] Ensure existing loans are not impacted&#x20;

<img src="../../.gitbook/assets/file.excalidraw (5).svg" alt="" class="gitbook-drawing">

## burn ATokens

* `burn` calls `_burnScaled`

<figure><img src="../../.gitbook/assets/image (69) (1).png" alt=""><figcaption></figcaption></figure>

Amount of tokens burnt is scaled against current liquidity index

* amount = 100
* liquidity index = 1.1
* `amountScaled` = 100 / 1.1 = 90

This is to account for interest accrued.&#x20;

{% hint style="info" %}
If you are confused regarding `balanceIncrease` and `_userState` see: [mint](../supply/transfer-and-mint.md#mint)
{% endhint %}

`_burnScaled` has an interesting approach, in that it looks to mint or burn the difference between unbooked interest accrued and the burn amount.

* unbooked interest accrued: interest accumulated since last update that has not been minted&#x20;

**If `balanceIncrease > amount`**

* interest accrued since last update is larger than burn amount
* mint the difference to the user as aTokens

**If `balanceIncrease <= amount`**

* burn amount exceeds interest accrued
* burn the difference from the user
