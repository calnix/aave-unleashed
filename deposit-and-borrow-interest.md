# Deposit & Borrow Interest

This section will cover interest rate calculations for both deposits and loans. The prerequisite is an understanding of simple(linear) interest and compound interest.

{% hint style="info" %}
Simple and compound interest: [Refresher ](appendix/simple-compound-apr-apy.md)
{% endhint %}

<figure><img src=".gitbook/assets/image (73) (2).png" alt=""><figcaption><p>_updateIndexes</p></figcaption></figure>

* **`calculateLinearInterest`** updates liquidity index => determines how deposit interest accumulates.
* **`calculateCompoundInterest`** updates variableBorrowIndex => determines how borrow interest accumulates.

## Deposit Interest

Supply interest in Aave accrues via linear interest (simple interest), as described in the function `calculateLinearInterest`.

<figure><img src=".gitbook/assets/image (103).png" alt=""><figcaption><p><code>_calculateLinearInterest</code></p></figcaption></figure>

{% hint style="info" %}
This function is executed within [`updateState`](functions/common-functions/.updatestate.md), which is called at the start of any state-changing function.&#x20;
{% endhint %}

The order of operations might be misleading; with a little re-arranging, it should be obvious:

<figure><img src=".gitbook/assets/image (144) (1).png" alt=""><figcaption><p>simplified <code>_calculateLinearInterest</code></p></figcaption></figure>

This should not be surprising to the reader; this is how simple interest is calculated.&#x20;

{% hint style="success" %}
* $$Simple\, Interest = P(r * T)$$&#x20;
* $$Total = P(1 + r * T)$$
{% endhint %}

However, the reader may wonder why `return 1e27 + result` - why add the ray at all?

This has to do with the calculation for `nextLiquidityIndex`:

<figure><img src=".gitbook/assets/image (172).png" alt="" width="563"><figcaption><p>nextLiquidityIndex</p></figcaption></figure>

Given that `nextLiquidityIndex` is calculated as seen above, we can conclude that the liquidity index is a multiplicative series, where the index begins with the value of 1:

$$
Index  = 1 * (1 + r_{1}t_{1})* (1+ r_{2}t_{2}) * (1+ r_{3}t_{3})...
$$

{% hint style="info" %}
Notice the similarities with $$Total = P(1 + r * T)$$.  Also, notice that the multiplicative series indicates compounding. The interplay between simple interest calculation and compounding is explained further down.&#x20;
{% endhint %}

### Example: calculating the next index

Given the details below, what would be the index at t1?

At t0:&#x20;

* liquidity index = 1.0
* supply interest rate, per second = 0.04

<figure><img src=".gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

The liquidity index at t1 would be 1.2 as a result of the accumulated interest over 5 seconds where the supply interest rate (per second) was 0.04.

### Compounding frequency

Let's connect the dots between interest and indexes by how they are calculated:

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption><p>Looks like simple interest!</p></figcaption></figure>

Given the manner in which deposit interest is calculated, it would seem that depositors on Aave do not enjoy the compounding of their interest.&#x20;

**This is not true.** Deposit interest does compound, but only **when the market is touched.**&#x20;

{% hint style="info" %}
**When the market is touched?**

* Indexes are updated when state-changing transactions are made: supply, borrow, repay, withdraw, etc.
* Each time the index is updated, interest is calculated for the interval from`lastUpdatedTimestamp` till now.&#x20;
* This is a simple interest calculation: $$(1 + rt)$$
* However, the new index is obtained by **multiplying** the prior index with $$(1 + rt)$$, which means that incoming interest is applied upon both the principal and previously accrued interest.

**Every time the index is updated, a new entry is created in the multiplicative series which contributes to compounding. However, the t in (1 + rt), for each period is irregular.**
{% endhint %}

**Let me illustrate with a 2-period example**

* User deposits 100 DAI at t0, when the Index is 1.0
* What is his balance at t2, when the Index has increased to 2.4

<figure><img src=".gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

From the breakdown on how Total is calculated it should be made apparent that due to the indexes being multiplied, interest is earned on both the principal and prior interest -> compound interest.

This is why earlier we mentioned that deposit interest compounds only when the market is touched -> index is updated, creating a new compounding period.&#x20;

{% hint style="info" %}
Compound interest: each period interest is generated on both the principal and the interest generated thus far.
{% endhint %}

## Borrow Interest

Borrow interest compounds every second. This is achiever by an approximation of binomial expansion to the third term.&#x20;

$$
Binomal\, Expansion: (1+x)^n =
 1+nx+\frac{n}{2}(n-1)x^2+\frac{n}{6}(n-1)(n-2)x^3+...
$$

Where n = t, number of periods and x = r, rate per second

$$
Interest:(1+r)^t \approx
 1+rt+\frac{t}{2}(t-1)r^2+\frac{t}{6}(t-1)(t-2)r^3
$$

For obvious reasons, the level of approximation has to be chosen such that the trade-off between accuracy and gas-savings is optimized.&#x20;

<figure><img src=".gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

**From** [**Aave-v2-whitepaper**](https://github.com/aave/protocol-v2/blob/master/aave-v2-whitepaper.pdf)**:**

* The function calculateCompoundedInterest, (MathUtils.sol line 46) implements the firrst three expansions which gives a good approximation of the compounded interest for up to a 5 year loan duration.&#x20;
* This results in a slight underpayment offset by the gas optimisation benefits.&#x20;
*   It's important to note that this behaves a little differently for variable and stable borrowing:&#x20;

    * For variable borrows, interests are accrued on any action of any borrower;&#x20;
    * For stable borrows, interests are accrued only when a specific borrower performs an action, increasing the impact of the approximation. Still, the difference seems reasonable given the savings in the cost of the transaction.



{% hint style="warning" %}
Deposit interest compounds only when the market is touched while borrow interest compounds every second due to an [additional calculation](https://github.com/aave/protocol-v2/blob/baeb455fad42d3160d571bd8d3a795948b72dd85/contracts/protocol/libraries/logic/ReserveLogic.sol#L359) performed in the contract.&#x20;
{% endhint %}

