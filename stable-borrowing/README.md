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

For new stable borrows the rate evolves based on utilization at the time. The stable interest rate$$R^s_t$$ follows the model:

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

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption><p>stable rate calculation</p></figcaption></figure>

</div>

In the picture above, the portions of `calculateInterestRates` that are pertinent to stable rate calculation have been extracted and illustrated. Feel free to compare with the code excerpt below.

**The following variables have their values set by Aave:**

* \_baseStableRateOffset
* slope1 and slope2 for both variable and stable instances
* OPTIMAL\_USAGE\_RATIO
* MAX\_EXCESS\_USAGE\_RATIO
* OPTIMAL\_STABLE\_TO\_TOTAL\_DEBT\_RATIO
* MAX\_EXCESS\_STABLE\_TO\_TOTAL\_DEBT\_RATIO

For a walkthrough on their values, please see the section on [DefaultReserveInterestRateStrategy](../contracts/defaultreserveinterestratestrategy.md).

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p> <code>calculateInterestRates</code> </p></figcaption></figure>

## Caveat: Rebalancing

If the protocol is in dire need of liquidity (e.g. prolonged liquidity shortage) and the supply APY is not sufficient to attract liquidity providers, to plug the liquidity shortage, some stable rate loans might undergo a procedure called **rebalancing**.&#x20;

If a user’s loan was taken at a stable rate that is deemed too low for the current market conditions, the protocol might decide to move the stable rate to the current (higher) one. This is known as rebalancing.&#x20;

{% hint style="info" %}
Rebalancing addresses the issue of stable borrowers enjoying an extremely discounted rate, in times of high utilization or low liquidity.&#x20;
{% endhint %}

Rebalancing allows the protocol to provide a more competitive supply rate for depositors. Eventually, as the market situation normalizes, the user will be rebalanced back down to a more appropriate stable rate.&#x20;

### Rebalancing condition

Rebalancing can occur when **depositors are earning** $$\leq$$ **90% of their earnings in a pure supply/demand market**.

* Treat existing stable debt as variable
* Given `totalVariableDebt` = (variable debt + stable debt), what would be the liquidity rate?
* If the current liquidity rate is $$\leq$$ 90% of the above calculated rate, rebalancing is valid

<figure><img src="../.gitbook/assets/image (185).png" alt=""><figcaption><p><a href="https://github.com/aave/aave-v3-core/blob/29ff9b9f89af7cd8255231bc5faf26c3ce0fb7ce/contracts/protocol/libraries/logic/ValidationLogic.sol#L394">validateRebalanceStableBorrowRate</a></p></figcaption></figure>

### Maximum size per stable borrow

Users will be able to borrow only a portion of the total available liquidity.

Aave enacts a max stable borrow limit for each stable borrow transaction. In each borrow transaction, if the borrow is a stable one, a check is conducted like so:

{% code title="validateBorrow()" overflow="wrap" %}
```solidity
vars.availableLiquidity = IERC20(params.asset).balanceOf(params.reserveCache.aTokenAddress);

//calculate the max available loan size in stable rate mode as a percentage of the
//available liquidity
uint256 maxLoanSizeStable = vars.availableLiquidity.percentMul(params.maxStableLoanPercent);

require(params.amount <= maxLoanSizeStable, Errors.AMOUNT_BIGGER_THAN_MAX_LOAN_SIZE_STABLE);
}
```
{% endcode %}

This ensures that the borrow amount set by the user is less than equal to `maxLoanSizeStable`, where `maxLoanSizeStable` is 25% of the deposited borrowable asset.

```solidity
maxStableLoanPercent = maxStableRateBorrowSizePercent (0.25e4)
// deposits are held on their respective aToken contract
availableLiquidity = total amt of borrowable asset deposited

maxLoanSizeStable = availableLiquidity * maxStableLoanPercent
		  = availableLiquidity * 0.25e4
```

{% hint style="info" %}
**If user wished to borrow DAI**

* DAI deposited into aDAI contract = 1000 DAI
* availableLiquidity = 1000 DAI
* maxLoanSizeStable = 250 DAI
{% endhint %}

## Implementation

We've talked about how the stable rate is determined on the whole, as the a reflection of the system state. We've also mentioned that user's lock-in the stable rate upon borrowing, less rebalancing conditions.&#x20;

Question you might have, is how does interest accrue for users, along with other implementation questions.&#x20;

**Global stable rate**

Each asset in Aave has its configuration data contained within a struct `ReserveData`. This can be accessed via an internal mapping `_reserves`.

Held in storage, each `ReserveData`, contains a variable `currentStableBorrowRate`.&#x20;

* This is the stable rate enjoyed by incoming stable borrows, regardless of user.
* This global value is determined by the Utilization model we described above.

**User's stable rate**

Upon taking up a stable loan, the rate enjoyed by the user is stored in `_userState[address].additionalData`, as defined on the stableDebtToken contract.

{% hint style="success" %}
We offered a constrained explanation here, following which you can see the [StableDebtToken](../contracts/stabledebttoken.md) section for specific code details.
{% endhint %}

##





{% hint style="warning" %}
Can I borrow using stable and variable rate at the same time for one asset?

* **NO**
* [https://docs.aave.com/faq/depositing-and-earning ](https://docs.aave.com/faq/depositing-and-earning)
{% endhint %}
