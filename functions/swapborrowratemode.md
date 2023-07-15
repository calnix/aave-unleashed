# swapBorrowRateMode

### Overview

This function is called when the user wishes to swap his current interest mode to the other:

* stable -> variable
* or vice versa

{% hint style="info" %}
Users can only have a debt position in **either** stable or variable. Cannot have both stable and variable debt for the same asset.
{% endhint %}

<figure><img src="../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache + updateState</mark>
* [ ] getUserCurrentDebt
* [ ] validateSwapRateMode
* [ ] Swap interest rate mode
* [ ] <mark style="color:orange;">updateInterestRates</mark>

We will breakdown and examine the unique sections of logics within **swapBorrowRateMode**. Code delineated in orange are common functions and can be explored in that [section](common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/YXcOdQXk6i8oUviCXrOx?darkMode=true" %}
