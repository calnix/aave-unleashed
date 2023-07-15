# Deposit & Borrow Interest

This section will cover interest rate calculations for both deposits and loans. The prerequisite is an understanding of simple(linear) interest and compound interest.

{% hint style="info" %}
Simple and compound interest: [Refresher ](appendix/simple-compound-apr-apy.md)
{% endhint %}

## Deposit Interest

Supply interest in Aave accrues via linear interest (simple interest), as described in the function `_calculateLinearInterest`.

<figure><img src=".gitbook/assets/image (103).png" alt=""><figcaption><p><code>_calculateLinearInterest</code></p></figcaption></figure>

Now the order of operations might be confusing, so with a little re-arranging, it should be obvious:

<figure><img src=".gitbook/assets/image (144).png" alt=""><figcaption><p>simplified <code>_calculateLinearInterest</code></p></figcaption></figure>

This should not be surprising to the reader; this is how simple interest is calculated.&#x20;

{% hint style="success" %}
* $$Simple\, Interest = P(r * T)$$&#x20;
* $$Total = P(1 + r * T)$$
{% endhint %}

However, the reader may wonder why `return 1e27 + result` - why add the ray at all?

This has to do with the calculation for `nextLiquidityIndex`:

<figure><img src=".gitbook/assets/image (172).png" alt="" width="563"><figcaption><p>nextLiquidityIndex</p></figcaption></figure>

Given that `nextLiquidityIndex` is calculated as such, we can conclude that the liquidity index is a multiplicative series, wherein the index beings with the value of 1 :

$$
Index  = 1 * (1 + r_{1}t_{1})* (1+ r_{2}t_{2}) * (1+ r_{3}t_{3})...
$$

* The index beings with 1
* The index always increases

{% hint style="info" %}
Recall what was reviewed in the earlier section on [Indexes](on-indexes/)
{% endhint %}

### Example: calculating the next index

Given the details below, what would be the index at t1?

* Index = 1.0, at t0
* supply rate, per second = 0.04

<figure><img src=".gitbook/assets/image (70).png" alt="" width="563"><figcaption></figcaption></figure>

The liquidity index at t1 would be 1.2 as a result of the accumulated interest over 5 seconds where the supply interest rate (per second) was 0.04.

### Compounding frequency

Given the manner in which deposit interest is calculated, it would seem that depositors on Aave do not enjoy the compounding of their interest.&#x20;

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption><p>Looks like simple interest!</p></figcaption></figure>

This is not true. Deposit interest does compound, but only [when the market is touched](https://github.com/aave/protocol-v2/blob/baeb455fad42d3160d571bd8d3a795948b72dd85/contracts/protocol/libraries/logic/ReserveLogic.sol#L349).&#x20;

Recall that previously we mentioned that indexes were updated when certain state-changing transactions were made: supply, borrow, repay, withdraw, etc.

These types of transactions cause a change in quantity of deposits or debts for a specific asset; as such the indexes and interest rates are updated to reflect the new state, post-transaction.

Each time index is updated, interest is calculated from the lastUpdatedTimestamp till now. This is a simple interest calculation.&#x20;

However, the new index is obtained by multiplying the prior index with $$(1 + rt)$$, which means that incoming interest is applied upon both the principal and previously accrued interest.

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

For obvious reasons, the level of approximation has to be chosen such that the tradeoff between accuracy and gas-savings is optimized.&#x20;

<figure><img src=".gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
Deposit interest compounds only [when the market is touched](https://github.com/aave/protocol-v2/blob/baeb455fad42d3160d571bd8d3a795948b72dd85/contracts/protocol/libraries/logic/ReserveLogic.sol#L349) while borrow interest compounds every second due to an [additional calculation](https://github.com/aave/protocol-v2/blob/baeb455fad42d3160d571bd8d3a795948b72dd85/contracts/protocol/libraries/logic/ReserveLogic.sol#L359) performed in the contract.&#x20;
{% endhint %}
