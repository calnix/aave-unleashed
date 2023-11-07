# borrow

## Overview

This function is called when the user wishes to take out a loan. Since Aave is an overcollateralized borrowing protocol, users can borrow up to a fraction of their deposited collateral.

{% hint style="info" %}
Before being able to call `borrow`, users must have deposited collateral via `supply`.
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw (19).svg" alt="" class="gitbook-drawing">

* `executeBorrow` contains the core logics of the borrow function

## executeBorrow

<figure><img src="../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache</mark>
* [ ] <mark style="color:orange;">updateState</mark>
* [ ] getIsolationMode
* [ ] validateBorrow
* [ ] mint debt token
* [ ] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

We will breakdown and examine the unique sections of logics within **executeBorrow**. Code delineated in orange are common functions and can be explored in that [section](../common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/gqyCixD4EOin748pMpq8?darkMode=true" %}
