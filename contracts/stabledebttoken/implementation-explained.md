# Implementation explained

## Global currentStableBorrowRate and users' local borrow rate

There is a global stable rate determined by a model; each user accrues interest based on the stable rate they locked-in. How does this work?

### **Global stable rate**

Each asset in Aave has its configuration data contained within a struct **`ReserveData`**. This can be accessed via an internal mapping **`_reserves`**.

Held in storage, each `ReserveData`, contains a uint128 variable `currentStableBorrowRate`.&#x20;

* This is the stable rate enjoyed by incoming stable borrows, regardless of user.
* This global value is determined by the Utilization model we described above.

{% hint style="info" %}
**Held in storage, the ReserveData struct contains variable: `currentStableBorrowRate`**
{% endhint %}

### **User's stable rate**

Upon taking up a stable loan, the rate enjoyed by the user is stored in `_userState[address].additionalData`, as defined on the stableDebtToken contract.

<figure><img src="../../.gitbook/assets/image (235).png" alt=""><figcaption><p>StableDebtToken is IncentivizedERC20</p></figcaption></figure>

* `.additionalData` is his weighted average stable rate, based on all his previous borrows.
* This is the rate at which interest compounds for the user.
* Each time a new stable borrow is taken, this is incremented:&#x20;
  * `(oldAmount * oldRate) + (newAmount * newRate) / totalAmount`

{% hint style="info" %}
**`_userState[address].additionalData`**` ``is updated within the`` `**`mint`**` ``function of stableDebtToken.`
{% endhint %}

### **\_avgStableRate**

An internal storage variable on the stable debt token contract, it is the weighted average rate across all the stable borrows taken to date, system-wide.&#x20;

Incremented like so:

**`_avgStableRate`**` ``= (currrentAvgStableRate * previousSupply) + (newRate * newAmount) / (_totalSupply() + newAmount)`

{% hint style="info" %}
Difference between **`_avgStableRate`**` ``and`` `**`currentStableBorrowRate`**

* **`_avgStableRate`**` ``reflects the average rate at which interest is being accrued by stable borrowers`
* **`currentStableBorrowRate`**` ``is the rate for the next incoming stable borrow as determined by the Utilization model.`
{% endhint %}

### **`borrow & mint`**

<figure><img src="../../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

`mint` returns the following two values which are stored in `reserveCache`:&#x20;

* `reserveCache.nextAvgStableBorrowRate` = `vars.currentAvgStableRate`&#x20;
* `reserveCache.nextTotalStableDebt` = `vars.nextSupply`

They are used later in **`calculateInterestRates`** to update system-wide rates.

### **updateInterestRates::calculateInterestRates**

After a loan is taken, **`calculateInterestRates`** is ran to update the system-wide rates:

* Liquidity rate
* Stable borrow rate
* Variable borrow rate

**`totalStableDebt`** used to calculate `totalDebt` and **`averageStableBorrowRate`** used to calculate `overallBorrowRate`, and consequently, `currentLiquidityRate`.

Stable borrow rate is updated, accounting for increase in stable loans taken; as are the other rates.&#x20;

{% hint style="success" %}
The updated stable rate is stored in **`reserve.currentStableBorrowRate - in storage.`**
{% endhint %}

### Visual Aid

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
