# Stable borrowing

The variable borrow rate has a per transaction resolution: it changes with each user interaction (borrow, deposit, withdraw, repayments or liquidations). This can result in significant volatility in the borrowing rate.&#x20;

To address this, and offer users a less volatile and more stable rate, stable borrowing was introduced. From tradfi markets, it is clear that there exists a segment of users that are willing to pay a premium for predictability; which is why fixed-income debt instruments, swap rate agreements and the like exist.&#x20;

{% hint style="info" %}
The stable rate provides predictability for the borrower; however, it comes at a premium, as the interest rates are higher than the variable rate.&#x20;
{% endhint %}

{% hint style="info" %}
Not all assets offer stable rates: assets that are most exposed to liquidity risk do not offer stable rates.
{% endhint %}

## Stable rate

Users who engage in stable borrowing will enjoy a fixed interest rate for the duration of their loan, with one caveat - see rebalancing condition below.

However, the stable rate for new loans varies on each interaction. Opening stable debt positions at different periods would likely result in different stable interest rates; due to differing liquidity state of the reserve pool. Ignoring rebalancing, these rates remain fixed for the lifetime of the position.

{% hint style="info" %}
The stable rate **for new loans** varies on each interaction.

Example:

* Alice and Bob take on stable loans at different points in time
* They enjoy stable rates of 4% and 6% respectively
* Different rates consequence of the differing liquidity state of the asset at that time.
* For the duration of their loans, their respective rates remains fixed.&#x20;
* However, in extreme liquidity conditions, their stable rates can be rebalanced to a higher rate; hence the nomenclature of **stable rate**, not fixed.
{% endhint %}

## Stable rate model

For new stables borrows the rate evolves based on utilization at the time. The stable interest rate$$R^s_t$$ follows the model:

$$if \hspace{1mm} U \leq U_{optimal}: \hspace{1cm} R^s_t = baseStableBorrowRate + R^s_{slope1}\frac{U_t}{U_{optimal}}$$

$$if \hspace{1mm} U > U_{optimal}: \hspace{1cm} R^s_t = baseStableBorrowRate + R^s_{slope1} + R^s_{slope2}\frac{U_t-U_{optimal}}{1-U_{optimal}}$$

* **baseStableBorrowRate** is $$R_{slope1} + premium$$
* $$R_{slope1}$$: variable rate slope1
* premium is a markup decided by Aave

The model adds an **additional premium** in the event that the `stableToTotalDebt` exceeds the optimal ratio of `stableToTotalDebt` as in the system.

$$
if \hspace{1mm} \frac{stableDebt_{t}}{TotalDebt} > \frac{stableDebt_{optimal}}{TotalDebt}: \hspace{0.3cm} R^s_t \,\, \texttt{+=} \,\,additionalPremium\frac{U^s_t-U^s_{optimal}}{1-U^s_{optimal}}
$$

* **additionalPremium** is similarly set like `baseStableBorrowRate`&#x20;

### Visual aid

To better understand the model and how it is implemented, we dissect `calculateInterestRates` on **DefaultReserveInterestRateStrategy**

<div data-full-width="false">

<figure><img src=".gitbook/assets/image (6) (1).png" alt=""><figcaption><p>stable rate calculation</p></figcaption></figure>

</div>

In the picture above, the portions of `calculateInterestRates` that are pertinent to stable rate calculation have been extracted and illustrated. Feel free to compare with the code excerpt below.

**The following variables have their values set by Aave:**

* \_baseStableRateOffset
* slope1 and slope2 for both variable and stable instances
* OPTIMAL\_USAGE\_RATIO
* MAX\_EXCESS\_USAGE\_RATIO
* OPTIMAL\_STABLE\_TO\_TOTAL\_DEBT\_RATIO
* MAX\_EXCESS\_STABLE\_TO\_TOTAL\_DEBT\_RATIO

For a walkthrough on their values, please see the section on [DefaultReserveInterestRateStrategy](contracts/defaultreserveinterestratestrategy.md).

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption><p> <code>calculateInterestRates</code> </p></figcaption></figure>

## Caveat: Rebalancing

If the protocol is in dire need of liquidity (e.g. prolonged liquidity shortage) and the supply APY is not high enough to satisfy the liquidity providers, some stable rate loans might undergo a procedure called **rebalancing**.&#x20;

If a user’s loan was taken at a stable rate that is too low for the current market conditions, the protocol might decide to move the stable rate to the current (higher) one to provide a more competitive supply rate for depositors.&#x20;

Eventually, as the market situation normalizes, the user will be rebalanced back down to a more appropriate stable rate.&#x20;

### Rebalancing condition

Rebalancing can occur when depositors are earning $$\leq$$ 90% of their earnings in a pure supply/demand market.

* Treat existing stable debt as variable
* Given `totalVariableDebt` = (variable debt + stable debt), what would be the liquidity rate?
* If the current liquidity rate is $$\leq$$ 90% of the above calculated rate, rebalancing is valid

<figure><img src=".gitbook/assets/image (185).png" alt=""><figcaption><p><a href="https://github.com/aave/aave-v3-core/blob/29ff9b9f89af7cd8255231bc5faf26c3ce0fb7ce/contracts/protocol/libraries/logic/ValidationLogic.sol#L394">validateRebalanceStableBorrowRate</a></p></figcaption></figure>



## WIP



## Incentive to rebalance

if your position is not so big you may never get rebalanced even if the rebalancing conditions are reached.&#x20;

{% hint style="info" %}
Rebalancing needs to be manually triggered by anyone, and there is more incentive to rebalance bigger positions rather than smaller, as that would affect rate more
{% endhint %}

{% hint style="warning" %}
Can I borrow using stable and variable rate at the same time for one asset? **NO**\
[**https://docs.aave.com/faq/depositing-and-earning** ](https://docs.aave.com/faq/depositing-and-earning)
{% endhint %}



## \_accrueToTreasury

```solidity
cumulatedStableInterest = currAvgStableBorrowRate CompoundInterest(stableLastUpdateTimestamp -> LastUpdateTimestamp)
prevTotalStableDebt = currPrincipalStableDebt * cumulatedStableInterest
```

Problem

* how is the model we see in DefaultReserveInterestRateStrategy connected to the issuance/buning of stableTokens seen in stableDebtToken
*



ReserveCache has&#x20;

* currPrincipalStableDebt
* currAvgStableBorrowRate
* currTotalStableDebt
* stableDebtLastUpdateTimestamp

These three are obtained via .**getSupplyData**() on the stableDebtToken contract:

<figure><img src=".gitbook/assets/image (161).png" alt=""><figcaption><p><strong>getSupplyData</strong></p></figcaption></figure>

* currPrincipalStableDebt => `super.totalSupply()`&#x20;
* currTotalStableDebt => `_calcTotalSupply(avgRate)`
* currAvgStableBorrowRate => `_avgStableRate`
* stableDebtLastUpdateTimestamp => `_totalSupplyTimestamp`



**Then**

* nextAvgStableBorrowRate = currTotalStableDebt&#x20;
* nexTotalStableDebt = currAvgStableBorrowRate&#x20;

## Connect defaultInterestRateStrategy to stableDebtToken

{% tabs %}
{% tab title="First Tab" %}


defaultInterestRate

* dictates how currentStableBorrowRate increments based on Utilziation

How does currentStableBorrowRate connect with a user's stable rate?

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



