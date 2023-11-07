# withdraw

## Overview

This function is called when the user wishes to withdraw their previously supplied asset. User redeems their aTokens for the underlying asset.&#x20;

* User gives aDAI, withdraws DAI

<img src="../../.gitbook/assets/file.excalidraw (18).svg" alt="" class="gitbook-drawing">

* `executeWithdraw` contains the core logics of the withdraw function

## executeWithdraw

<figure><img src="../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache</mark>
* [ ] <mark style="color:orange;">updateState</mark>
* [ ] get user's AToken balance + amount to withdraw
* [ ] validateWithdraw: confirm user has sufficient balance and asset isActive
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] collateral check
* [ ] burn ATokens
* [ ] Ensure existing loans are not impacted&#x20;

We will breakdown and examine the unique sections of logics within **withdraw**. Code delineated in orange are common functions and can be explored in that [section](../common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/vSySjOVTyDhS6fxi6Tt4?darkMode=true" fullWidth="false" %}

