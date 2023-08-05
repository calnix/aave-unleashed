---
description: https://docs.aave.com/developers/v/2.0/the-core-protocol/debt-tokens
---

# StableDebtToken

We will examine stableDebtToken contract in this section.

## getSupplyData

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption><p><strong>getSupplyData</strong></p></figcaption></figure>

This function is called in .cache, and we will explain each component.&#x20;

{% code title=".cache(...)" %}
```solidity
(
  reserveCache.currPrincipalStableDebt,
  reserveCache.currTotalStableDebt,
  reserveCache.currAvgStableBorrowRate,
  reserveCache.stableDebtLastUpdateTimestamp
) = IStableDebtToken(reserveCache.stableDebtTokenAddress).getSupplyData();
```
{% endcode %}

### super.totalSupply()

* Inherited from IncentivizedERC20.sol
* Essentially a getter function for the internal storage variable `_totalSupply`

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>IncentivizedERC20.sol</p></figcaption></figure>

{% hint style="info" %}
On \_totalSupply:

* Reflects total stable debt, accounting for interest accrued from inception till **`_totalSupplyTimestamp`.**
* Does not account for interest accrued from **`_totalSupplyTimestamp`**` ``to now.`
{% endhint %}

### \_totalSupply

Represents total stable debt, accounting for interest accrued since inception till  `_totalSupplyTimestamp`**.**&#x20;

`_totalSupply` is incremented on `mint`, decremented on `burn`.

**Example:** a stable borrow is taken some time after `_totalSupplyTimestamp`&#x20;

* `_totalSupply += stableBorrowAmount + unbooked interest`&#x20;

`_totalSupply` is incremented to account for the incoming borrow as well as the interest accrued in the period since `_totalSupplyTimestamp`**.**

{% hint style="info" %}
Assigned to `currPrincipalStableDebt, in .cache`
{% endhint %}

{% hint style="info" %}
* `_totalSupply` is declared in `IncentivizedERC20`
* `_totalSupplyTimestamp` is declared in `StableDebtToken`
{% endhint %}

### \_calcTotalSupply(avgRate)

Calculates total stable debt, accounting for interest accrued to date.&#x20;

* `avgRate` is `_avgStableRate`
* `principalSupply is _totalSupply`

<figure><img src="../.gitbook/assets/image (9) (2).png" alt=""><figcaption></figcaption></figure>

* calculates interest compounded from `_totalSupplyTimestamp` till now (`block.timestamp`)
* `_totalSupplyTimestamp`: Timestamp of the last update of \_totalSupply
  * updated in `mint` & `burn`

{% hint style="info" %}
Assigned to `currTotalStableDebt`, in .cache
{% endhint %}

**Why do we calculate interest from `_totalSupplyTimestamp`?**

Every time `mint` or `burn` is called, `_totalSupply` is updated such that it accounts for the interest accrued since previous update till now, as well as the `mint`/`burn` amount.&#x20;

For example, at the start of `mint`

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

From this, we can see that `_totalSupply` is updated with the interest accrual from previous timestamp till now.

Since this occurs on each function call that would modify `_totalSupply`, the definition of principalStableDebt is to distinguish from accounted interest and floating, unaccounted interest.&#x20;

### \_avgStableRate

* internal storage variable
* weighted average rate, calculated across all stable borrows

<figure><img src="../.gitbook/assets/image (4) (1) (3).png" alt=""><figcaption></figcaption></figure>

Simply put, assume there are 3 stable borrows at differing times:

* 1: 100 DAI at 1%
* 2: 200 DAI at 2%
* 3: 300 DAI at 3%

weighted average rate = (100 \* 1%) + (200 \* 2%) + (300 \* 3%) / (100 + 200 + 300) = **2.3%**

**\_avgStableRate = 2.3%**

{% hint style="info" %}
**ReserveCache contains the following:**&#x20;

* currPrincipalStableDebt => `super.totalSupply()`&#x20;
* currTotalStableDebt => `_calcTotalSupply(avgRate)`
* currAvgStableBorrowRate => `_avgStableRate`
* stableDebtLastUpdateTimestamp => `_totalSupplyTimestamp`

**`Also`**

* nextAvgStableBorrowRate = currTotalStableDebt&#x20;
* nexTotalStableDebt = currAvgStableBorrowRate&#x20;
{% endhint %}

## balanceOf

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The balance for any address is calculated to account for interest accrued since the last interaction.

{% hint style="info" %}
* each user's stable rate is stored at `_user[account].additionalData`&#x20;
* `_timestamps[account]` stores the timestamp of their last interaction
{% endhint %}

## totalSupply()

Declared on StableDebtToken.sol.

<figure><img src="../.gitbook/assets/image (3) (3).png" alt=""><figcaption></figcaption></figure>

**`_calcTotalSupply(_avgStableRate)`**

* [`super.TotalSupply()`](stabledebttoken.md#super.totalsupply)  returns `_totalSupply`; accounts for interest from inception till `_totalSupplyTimestamp`.
* [`_calcTotalSupply`](stabledebttoken.md#\_calctotalsupply-avgrate) will compound this with the recently accrued interest, from `_totalSupplyTimestamp` till now.

Therefore, `totalSupply` returns the total stable debt, accounting for all interest to date.&#x20;

## mint

Let's examine mint, from the pretext that is has been called via `executeBorrow`.

`mint` is called via the interface `IStableDebtToken`, `reserve.currentStableBorrowRate` is passed as a param.

* variable is cached to avoid unnecessary calls to storage: `currentStableRate`

<figure><img src="../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

#### **\_calculateBalanceIncrease**&#x20;

calculates the increase in balance due to compounding interest, for a specific user, since the previous &#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### **Update \_totalSupply**&#x20;

```solidity
vars.previousSupply = totalSupply();
vars.currentAvgStableRate = _avgStableRate;
vars.nextSupply = _totalSupply = vars.previousSupply + amount;
```

* `_totalSupply` is updated to be `previousSupply + amount`
* `previousSupply` reflects total stable debt and recently accrued interest as explained in [totalSupply](stabledebttoken.md#totalsupply)
* hence, `_totalSupply` is incremented to account for both unbooked interest and incoming borrow.

#### **Calculate nextStableRate**

`reserve.currentStableBorrowRate` is passed as `rate`.

{% code overflow="wrap" %}
```solidity
vars.currentStableRate = _userState[onBehalfOf].additionalData;
vars.nextStableRate = (vars.currentStableRate.rayMul(currentBalance.wadToRay()) + vars.amountInRay.rayMul(rate)).rayDiv((currentBalance + amount).wadToRay());

_userState[onBehalfOf].additionalData = vars.nextStableRate.toUint128();
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

* From now till the next future interaction, interest will compound at the nextStableRate.
* This is reflected in [balanceOf](stabledebttoken.md#balanceof), in **\_calculateBalanceIncrease** section.

#### **Calculate updated average stable rate**

```solidity
// Calculates the updated average stable rate
vars.currentAvgStableRate = 
_avgStableRate = (
  (vars.currentAvgStableRate.rayMul(vars.previousSupply.wadToRay()) +  rate.rayMul(vars.amountInRay)).rayDiv(vars.nextSupply.wadToRay())
).toUint128();

```

<figure><img src="../.gitbook/assets/image (6) (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

#### \_mint&#x20;

<figure><img src="../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

* increments user's balance by amount
* makes a call to \_incentivesController, should it be defined

{% hint style="info" %}
The maximum 128-bit integer, $$2^{128}− 1$$is a number that is not usefully written out in words or in all of its 39 digits: 340282366920938463463374607431768211455

Here it is in words:

> three hundred forty undecillion, two hundred eighty-two decillion, three hundred sixty-six nonillion, nine hundred twenty octillion, nine hundred thirty-eight septillion, four hundred sixty-three sextillion, four hundred sixty-three quintillion, three hundred seventy-four quadrillion, six hundred seven trillion, four hundred thirty-one billion, seven hundred sixty-eight million, two hundred eleven thousand, four hundred fifty-five
{% endhint %}

#### Return variables

* nextSupply
* currentAvgStableRate

### Visual Aid

<img src="../.gitbook/assets/file.excalidraw (1) (1) (1).svg" alt="" class="gitbook-drawing">

## burn

This function is typically called through [repay](../functions/repay/), when the user wishes to repay all or some of his stable debt.

<img src="../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

### **Get variables**

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

**\_calculateBalanceIncrease**

* `currentBalance` accounts for latest interest, updated to `block.timestamp`
* `balanceIncrease`: increase in interest, acrrued between last update and now

**previousSupply = totalSupply()**

* get total supply of stable debt
* calculated via `_calcTotalSupply(_avgStableRate)`

**get user's stable rate:** `_userState[from].additionalData`

### **Decrement avgStable rate accordingly**

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

**if `(totalSupply <= amount)`:**

A discrepancy arises, such that there is no debt to repay; this is possible because total debt accrues seperately from each user's individual debt.&#x20;

* if there is discrepancy, simply reset total supply & avgStableRate to 0.
* user's debt, which he try repaying is the remainding global debt.

**`Else`:**

* update `_totalSupply = previousSupply - amount` \[store in local variable `nextSupply`]

{% hint style="warning" %}
**Discrepancy:**

* Similar to before, there might be a discrepancy arising due to global rate and user's rate being tracking independently.
* To identify if such a discrepancy exists: \
  if **`userRate * userBalance > avgRate * totalSupply`**, reset both totalSupply and avgStableRate.
* note that **`totalSupply`** has been updated, less the amount to burn
{% endhint %}

* Otherwise, update avgRate as per `nextAvgStableRate` calculation.&#x20;

{% hint style="info" %}
`nextAvgStableRate`: \[(avgRate \* totalSupply) - (userRate \* userBalance)] / (totalSupply - amount)
{% endhint %}

### Update user info

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

**if amount == user's updated balance:**

* reset user's stabel rate
* reset user's lastUpdatedTimestamp -> no more debt. repaid in full. else:
* just update user's lastUpdatedTimestamp

{% hint style="info" %}
Global `_totalSupplyTimestamp` is updated as well
{% endhint %}

### \_mint or \_burn&#x20;

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Depends if accrued interest > user input

* if(balanceIncrease > amount): mint the difference&#x20;
* else: burn the difference
