# repay

## Overview

This function is called when the user wishes to repay his existing debts. User can opt to repay in full or partially.

<img src="../../.gitbook/assets/file.excalidraw (19).svg" alt="" class="gitbook-drawing">

**Inputs**

* address of debt asset: if repaying DAI debt, pass the DAI address
* amount of debt to repay: does not need to be 100%
* interestRateMode: stable or variable
* onBehalfOf: if repaying another user's debt
* useATokens: false

{% hint style="info" %}
**`repayWithATokens`** has `useATokens`: `true`
{% endhint %}

## executeRepay

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache + updateState</mark>
* [ ] get current debt&#x20;
* [ ] validateRepay
* [ ] get paybackAmount
* [ ] burn debt tokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] setBorrowing
* [ ] updateIsolatedDebtIfIsolated
* [ ] handle repayment

We will breakdown and examine the unique sections of logics within **executeRepay**. Code delineated in orange are common functions and can be explored in that [section](../common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/7o8NGeqZaTRyhtCcaHmb?darkMode=true" %}
