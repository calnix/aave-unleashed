---
description: https://docs.aave.com/developers/v/2.0/the-core-protocol/debt-tokens
---

# StableDebtToken



We will examine stableDebtToken contract in this section.

## getSupplyData

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption><p><strong>getSupplyData</strong></p></figcaption></figure>

### super.totalSupply()

* Inherited from IncentivizedERC20.sol

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>IncentivizedERC20.sol</p></figcaption></figure>

* returns internal storage variable `_totalSupply`
* `_totalSupply` is incremented on `mint`, decremented on `burn`
* on borrow: `_totalSupply = _totalSupply + amount`

{% hint style="info" %}
* Accounts for interest accrued in past periods.&#x20;
* Does not account for interest accrued since **`_totalSupplyTimestamp`**
* treated as `currPrincipalStableDebt`
{% endhint %}

### \_calcTotalSupply(avgRate)

Reflects total stable debt, where avgRate is `_avgStableRate`.

* `totalSupply = principalSupply * cumulatedInterest`
* `principalSupply is _totalSupply`

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* calculates interest compounded from `_totalSupplyTimestamp` till now (`block.timestamp`)
* `_totalSupplyTimestamp`: Timestamp of the last update of the total supply
  * updated in `mint` & `burn`

**Why do we only calculate compound interest from `_totalSupplyTimestamp`?**

Every time `mint` or `burn` is called, `_totalSupply` is updated such that it accounts for the interest accrued since previous update till now, as well as the `mint`/`burn` amount.&#x20;

For example, at the start of `mint`

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

From this, we can see that `_totalSupply` is updated with the interest accrual from previous timestamp till now.

Since this occurs on each function call that would modify `_totalSupply`, the definition of principalStableDebt is to distinguish from accounted interest and floating, unaccounted interest.&#x20;

### \_avgStableRate

* internal storage variable
* weighted average rate, calculated across all stable borrows

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

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
{% endhint %}





**Then**

* nextAvgStableBorrowRate = currTotalStableDebt&#x20;
* nexTotalStableDebt = currAvgStableBorrowRate&#x20;









## \_accrueToTreasury

```solidity
cumulatedStableInterest = currAvgStableBorrowRate CompoundInterest(stableLastUpdateTimestamp -> LastUpdateTimestamp)
prevTotalStableDebt = currPrincipalStableDebt * cumulatedStableInterest
```

Problem

* how is the model we see in DefaultReserveInterestRateStrategy connected to the issuance/burning of stableTokens seen in stableDebtToken
*



## Connect defaultInterestRateStrategy to stableDebtToken

{% tabs %}
{% tab title="First Tab" %}


defaultInterestRate

* dictates how currentStableBorrowRate increments based on Utilization

**How does currentStableBorrowRate connect with a user's stable rate?**

* stored in reserveData
* updated when .updateRates::calculateInterestRates

::WITHIN CACHE: reserveCache holds

* currPrincipalStableDebt
* currTotalStableDebt
* currAvgStableBorrowRate
* stableDebtLastUpdateTimestamp -----------------------------------> getSupplyData
* nextTotalStableDebt = currTotalStableDebt
* nextAvgStableBorrowRate = currAvgStableBorrowRate

THIS IS THE CONNECTION: on borrowing stable,

* currentStableRate = reserve.currentStableBorrowRate
* stableToken.mint(user, onbehalfOf, amount, currentStableRate)

### the mint fn uses currentStableRate as follows:

* update \_avgStableRate
{% endtab %}

{% tab title="Second Tab" %}


1. cache reserveCache holds
2. currPrincipalStableDebt
3. currTotalStableDebt
4. currAvgStableBorrowRate
5. stableDebtLastUpdateTimestamp -----------------------------------> getSupplyData
6. nextTotalStableDebt = currTotalStableDebt
7. nextAvgStableBorrowRate = currAvgStableBorrowRate
8. borrow stable on borrowing stable:
9. stableToken.mint(user, onbehalfOf, amount, reserve.currentStableBorrowRate)
10. rate = currentStableBorrowRate

// the mint fn uses currentStableBorrowRate as follows: // calculating user's nextStableRate currentStableRate = \_userState\[borrower].additionalData nextStableRate = \[currentStableRate \* (currentBalance + amountInRay\*rate)] / (currentBalance + amount) \_userState\[borrower].additionalData = nextStableRate

//update \_avgStableRate currrentAvgStableRate = \_avgStableRate = = currrentAvgStableRate \* (previousSupply + rate\*amount) / nextSupply

return currrentAvgStableRate == reserveCache.nextAvgStableBorrowRate

3. Update Rate

averageStableBorrowRate == reserveCache.nextAvgStableBorrowRate

* used to calc overall borrow rate to get liquidity rate

\================================================================================================ currAvgStableBorrowRate currPrincipalStableDebt

* derived from getSupplyData
{% endtab %}
{% endtabs %}



