---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# On Indexes

Now that we understand how interest rates are modelled, let's look at how on-chain interest accrual works. Central to this is the concept of indexes.

Aave uses indexes to track interest accumulation. This is crucial in the context of a blockchain, where events are tracked not by timestamps, but by blocks.

There are two types of indexes in Aave:

1. **Liquidity Index**: tracks supply interest accrued from inception to date
2. **Variable Borrow Index**: tracks variable borrow interest from inception to date

Both indexes represent the cumulative growth of interest over time for a specific asset, and continually increment over time.

{% hint style="info" %}
An index cannot decrement as cumulative interest always increases.&#x20;

* Interest always adds up over time, regardless of how interest rates themselves may vary.&#x20;
{% endhint %}

{% hint style="warning" %}
This positive relationship between indexes and time can be broken if we somehow introduced negative interest rates into Defi.&#x20;

This has been talked about in mainstream economics, with central banks experimenting with negative rates to stimulate growth. Hopefully, it will not come to pass.&#x20;
{% endhint %}

## How does it work?&#x20;

Instead of updating the index every block, Aave timestamps specific values of the index during state-changing transactions.

An index starts with a value of 1.0 at inception. As time passes and interest accrues, the liquidity index increases. For example, after a certain period, the liquidity index might be 1.05, indicating that 5% interest has been accrued on all deposits specific to that asset.

* Indexes start at 1.0
* Indexes always increase over time

{% hint style="info" %}
This interest comes from borrowers who are paying interest on their loans.
{% endhint %}

### Index calculation

To calculate a new index value, the old index value is multiplied by the interest rate and the elapsed time.&#x20;

<figure><img src="../.gitbook/assets/image (147).png" alt="" width="420"><figcaption><p>Index formula</p></figcaption></figure>

#### **Assume**&#x20;

* ETH has just been made available on Aave for lending and borrowing - liquidity index starts at  **1.0**.&#x20;
* We have a user who initially deposits 10 ETH into the Aave lending pool.&#x20;

Now, let's consider the following changes in interest rates and their respective time periods:

| Month   | Interest Rate, annual |
| ------- | --------------------- |
| Month 1 | 12%                   |
| Month 2 | 6%                    |
| Month 3 | 8%                    |

To calculate the liquidity index at each month, we multiply the previous index by the factor `(1 + interest rate * time elapsed)`. Here's how the liquidity index evolves:

| Month          | Liquidity Index                       |
| -------------- | ------------------------------------- |
| End of Month 1 | 1 \* (1 + 0.12 \* 1/12) = 1.01        |
| End of Month 2 | 1.01 \* (1 + 0.06 \* 1/12) = 1.0105   |
| End of Month 3 | 1.0105 \* (1 + 0.08 \* 1/12) = 1.0113 |

Based on the evolving liquidity index, we can calculate how the user's deposit grows over time. Starting with 10 ETH, here's the growth of the user's deposit:

| Month          | Deposit                                  |
| -------------- | ---------------------------------------- |
| End of Month 1 | 10 ETH \* 1.01 = 10.1 ETH                |
| End of Month 2 | 10.1 ETH \* 1.0105 = 10.20105 ETH        |
| End of Month 3 | 10.20105 ETH \* 1.0113 = 10.31344665 ETH |

As the interest rates change over time, the liquidity index updates, and the user's deposit grows accordingly. The deposit value increases each month based on the compounded interest earned from the changing interest rates and the corresponding time periods.

{% hint style="info" %}
Whenever a state-changing function is called (supply, withdraw, borrow, repay, etc) indexes are updated. Hence, indexes are said to be updated on user interaction, in a non-periodic manner.&#x20;
{% endhint %}

### One Index to rule them all&#x20;

To accurately value each user's deposit and loan positions within the system, we will need to track the timestamps of all the instances of:

* deposits, withdrawals,&#x20;
* borrows taken, borrows repaid (in full or partial)

Tracking, calculating and storing each individual's balances by initializing indexes at a user-level would be both gas and storage intensive.&#x20;

Hence, the streamlined approach is to have two global indexes, one to track deposit interest from inception and another to track borrow interest.&#x20;

Let me give an analogy. Imagine a hotel that continually adds floors to its building. The hotel represents the liquidity index (increasing deposit interest).

* each floor added represents additional interest accumulated&#x20;
* a new depositor checks-in at the highest floor&#x20;
* his interest earned is represented by the floors added above his entry level

However, w.r.t to the floors **below** his arrival, that is interest accumulated prior to his deposit, of which he has no claim. Hence, users' deposits and withdrawals are scaled against the indexes - to discount prior interest.&#x20;

Let me illustrate.

#### Scaling Positions

Deposits in Aave are scaled down against the current liquidity index, at the time of deposit. Like so:

> amountScaled = Deposit Amount / Current Liquidity Index

In the same vein, existing balances are scaled up with liquidity index:

> Balance = amountScaled \* Current Liquidity Index

Since the liquidity index serves to reflect the interest accumulated since inception, deposits are divided against the liquidity index at time of deposit to negate all prior interest.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* User opts to deposit at t1, when Index = 1.1
* His deposit is scaled down by dividing against the index, to negate all interest prior to t1
* Should he choose to withdraw at t2, his scaled balance is scaled up by the index then, which accumulates **all** interest since inception, t0.

{% hint style="info" %}
We will talk about `_userState[account].balance` when we dive deeper into ATokens and how exactly user balances are stored and retrieved.&#x20;
{% endhint %}

{% hint style="success" %}
* Deposit interest is distributed across all depositors proportional to their balance and time committed.&#x20;
* A single index is used to track the accumulated interest for everyone.&#x20;
* The index and rate are updated for every deposit, borrow, withdrawal, repay of ANY Aave user.&#x20;
* The update works like this: Utilization Rate -> Variable Borrow rate -> Liquidity rate -> Liquidity Index
{% endhint %}
