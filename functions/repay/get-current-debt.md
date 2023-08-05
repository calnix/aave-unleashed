# get current debt

## Overview

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache + updateState</mark>
* [ ] get current debt&#x20;
* [ ] validateRepay
* [ ] get paybackAmount
* [ ] burn debt tokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] setBorrowing
* [ ] updateIsolatedDebtIfIsolated
* [ ] handle repayment

## get current debt

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

* queries both the stable debt and variable debt token balance for the target user
* For a specific asset user can only borrow in either stable or variable mode; cannot have concurrent debt in both modes simultaneously
* Therefore, one of the values returned will be `0`

{% hint style="info" %}
`balanceOf` returns the debt balance accounting for interest up till the current block.timestamp
{% endhint %}
