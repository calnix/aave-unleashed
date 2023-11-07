# validateRepay, paybackAmount

## Overview

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache + updateState</mark>
* [x] get current debt&#x20;
* [ ] **validateRepay**
* [ ] **get paybackAmount**
* [ ] burn debt tokens
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] setBorrowing
* [ ] updateIsolatedDebtIfIsolated
* [ ] handle repayment

## validateRepay

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

* `amountSent` is `params.amount`, which is `amount` passed into `repay`

<figure><img src="../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

**The first two require statements serves as input validation for `amountSent`**

1. ensure that value is non-zero
2. ensure that an explicit amount was set by the user, **else** ensure that msg.sender is paying for himself

{% hint style="info" %}
1. If `amountSent` is `type().max` -> indicates an explicit amount was not set by user
2. `msg.sender` **is not** `onBehalfOf`: paying for someone else&#x20;
3. when repaying for another party, user must be **explicit** in amount&#x20;
   * cannot just whack uint256.max as a cover all&#x20;
   * cos' what if the debt was unexpectedly large
{% endhint %}

**Then we check if the asset status via `getFlags`:**

* Active
* Not Paused

{% hint style="info" %}
See link for in-depth explanation on [**getFlags**](../common-functions/getflags/)
{% endhint %}

**Lastly, we check that user has a non-zero debt in either one of the interest rate mode: stable or variable.**

* if both are zero, there is no debt to repay

## Setting paybackAmount

<figure><img src="../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

* Select the non-zero debt value and assign it to **`paybackAmount`**

{% hint style="info" %}
Here **`paybackAmount`** reflects the total debt position of the user.
{% endhint %}

### **`If useAtokens == true && params.amount == type.max`**&#x20;

* overwrite repay amount (`params.amount`) to the user's AToken balance&#x20;
* this is the max that can be repaid, given AToken balances

{% hint style="info" %}
Applicable for `repayWithATokens`
{% endhint %}

In the case of `repay`, `useATokens` is set to **`false`**, therefore the code within the if section is not executed.

### **`If params.amount < paybackAmount`**&#x20;

* User has opted to repay his debt partially,&#x20;
* overwrite **`paybackAmount`** to reflect user's choice of partial repayment

{% hint style="info" %}
Here **`paybackAmount`** reflects what the user wishes to repay.
{% endhint %}

{% hint style="success" %}
Another way to view this, is: "Is the user able to cover his total debt with the repay amount, if not, set `paybackAmount` to what he has offered to repay"
{% endhint %}
