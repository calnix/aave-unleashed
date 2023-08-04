---
description: https://docs.aave.com/risk/asset-risk/risk-parameters#liquidation-threshold
---

# Liquidation

## TLDR

If a user's health factor falls bellow 1.0, he is subject to liquidation.

* user’s health factor: $$0.95 < hf < 1$$, the loan is eligible for a liquidation of 50%.
* user’s health factor: $$hf <= 0.95$$, the loan is eligible for a liquidation of 100%.

The user's max repayable bad debt is determined by close factor (50%/100%), which is a function of his health factor.

Liquidators are incentivised to hunt bad loans by being awarded a liquidation bonus. This bonus is financed by the liquidation penalty suffered by users who fail to keep their loans healthy.

The protocol levies a small liquidation fee on successful liquidations - a cut from the liquidation bonus liquidators enjoy.

{% hint style="success" %}
**User - on liquidation**

* suffers liquidation penalty
* loses collateral equivalent to the value of repaid debt + liq. bonus&#x20;
* **collateral lost = (repay + liq. bonus)**

**Liquidator**

* gains liquidation bonus from user
* pays liquidation protocol fee
* **profit = (liquidation bonus - protocol fee - gas)**
{% endhint %}

{% hint style="warning" %}
Liquidators must already have sufficient balance of the debt asset, which will be used by the `liquidationCall` to pay back the debt. They can use `flashLoan` for liquidations.
{% endhint %}

## Overview

#### **There are two ways that a loan can become unhealthy:**&#x20;

1. collateral asset(s) provided could decrease in value,&#x20;
2. borrowed debt could increase in value against the collateral provided, or the user could fail to keep up with their interest payments.&#x20;

Conversely, a user’s health factor can be directly improved by either paying off part of the debt or by offering more collateral.&#x20;

{% hint style="success" %}
Managing an on-chain debt position is very similar to managing a leveraged trading position due the requirements of overcollateralization and liquidation.&#x20;
{% endhint %}

#### **What happens to an unhealthy loan?**

It gets liquidated.&#x20;

The liquidation mechanism protects lenders from the risk of default and maintains the solvency of the lending platform. It incentivizes borrowers to maintain sufficient collateralization and encourages active participation from liquidators who can profit from liquidating unhealthy positions.

**Liquidation process**

* Borrowers must maintain a **minimum health factor**, which compares their total collateral value to their outstanding loan value.&#x20;
* If borrower's loan position falls below the minimum health factor, their position becomes eligible for liquidation.
* During the liquidation, the borrower's collateral is seized and sold in the open market to repay the outstanding loan.
* A liquidation penalty is applied to discourage borrowers from allowing their position to become unhealthy.
* Liquidators are offered incentives to liquidate unhealthy positions; this is financed by the liquidation penalty.&#x20;

**Example**

* Bob deposits 5 ETH and 4 ETH worth of YFI, and borrows 5 ETH worth of DAI&#x20;
*  If Bob’s Health Factor drops below 1 his loan will be eligible for liquidation.&#x20;
* A liquidator can repay up to 50% of a single borrowed amount = **2.5 ETH worth of DAI**.
* In return, the liquidator **can claim a single collateral;** as the liquidation bonus is higher for YFI (15%) than ETH (5%) the liquidator chooses to claim YFI. &#x20;
* Liquidation incentive: 15% \* 2.5 ETH = 0.375 ETH
* Liquidator claims 2.5 + 0.375 ETH worth of YFI for repaying 2.5 ETH worth of DAI.

{% hint style="info" %}
* user’s health factor: $$0.95 < hf < 1$$, the loan is eligible for a liquidation of 50%.
* user’s health factor: $$hf <= 0.95$$, the loan is eligible for a liquidation of 100%.
{% endhint %}

### Loan To Value: LTV

* Defines the maximum amount of assets that can be borrowed with a specific collateral.
* Expressed as a percentage.
* Example:
  * ETH has an LTV of 75%&#x20;
  * For every 1 ETH worth of collateral, can borrow 0.75 ETH worth of some asset.

{% hint style="warning" %}
LTV is always < 100% => Aave operates on overcollateralized lending.
{% endhint %}

#### Average LTV

For a user with multiple assets as collaterals, his average LTV would be calculated as such:

$$
Avg \: LTV = \frac{ \sum{[Collateral_i \: ($)   \: \times \: LTV_i}]}{Total \: Collateral  \:($) \: \:}
$$

Note that we need to normalize the valuation of different assets against a common base currency. In the example above, it is expressed as dollars; but it could similarly by valuing all the different collateral amounts in ETH.

{% hint style="info" %}
Each market has an AaveOracle contract where you can query token prices in the base currency (ETH on V2 mainnet/polygon and USD on all other markets)
{% endhint %}

### **Liquidation Threshold**

* The liquidation threshold is the percentage at which a debt position is defined as undercollateralized.&#x20;
* A Liquidation threshold of 80% means that if the loan value rises above 80% of the collateral value, the position is undercollateralized and can be liquidated.
* The delta between the LTV and the Liquidation Threshold is a safety mechanism in place for borrowers.

{% hint style="info" %}
**Example**&#x20;

LTV at 70% and a liquidation threshold of 75% means that users have a 5% margin of safety.

* can maximally borrow up to 70% of collateral value.
* will be liquidated if loan value increases to  75% of collateral value.
* 5% safety margin to endure price volatility.&#x20;
{% endhint %}

#### Average Liquidation Threshold

For a user with multiple assets as collaterals, his average LTV would be calculated as such:

$$
Avg\, Liquidation\, Threshold = \frac{\sum{[Collateral_i \, ($)  \times Liq.threshold_i]}}{Total \: Collateral  \:($)}
$$

Note that we need to normalize the valuation of different assets against a common base currency.&#x20;

{% hint style="info" %}
For each wallet, the Liquidation Threshold is calculate as the weighted average of the Liquidation Thresholds of the collateral assets and their value:

$$Liquidation \: Threshold= \frac{ \sum{Collateral_i \: in \: ETH \: \times \: Liquidation \: Threshold_i}}{Total \: Collateral \: in \: ETH \:}$$
{% endhint %}

### **Health factor**

The health factor is the numeric representation of the safety of your collateral against the borrowed assets. The higher the value is, the safer the state of your funds are against a liquidation scenario.&#x20;

* Wallets with Health Factor below 1 can get liquidated.
* The health factor depends on the liquidation threshold of your collateral against the value of your borrowed funds.

{% hint style="info" %}
Think margin calls on leveraged positions.&#x20;
{% endhint %}

Account health is calculated as the ratio of risk-adjusted collateral value to liabilities. The health factor of an account with a single loan position is calculated as follows

$$
HF = \frac{collateral \, ($) * Liq.threshold}{Debt \,($)}
$$

{% hint style="info" %}
For each wallet, these risks parameters enable the calculation of the health factor:

$$H_f = \frac{ \sum{Collateral_i \: in \: ETH \: \times \: Liquidation \: Threshold_i}}{Total \: Borrows \: in \: ETH}$$

When $$H_f < 1$$ the position may be liquidated to maintain solvency.
{% endhint %}

#### Example: Putting it all together

<figure><img src="../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

**Initially, when ETH/USDT = $4000,**

* User posts **0.25 ETH** as collateral; collateral is valued at **$1000**.
* The max debt possible is **$700** \[$$1000 * 0.70$$] of any asset; given LTV of ETH is 70%.
* Assume he borrows **$500** of some arbitrary asset.
* Account health is **1.5** \[$$\frac{1000 * 0.75}{500}$$]; given liquidation threshold of 75%.

**Some time after, when ETH/USDT drop to $2664,**

* Value of collateral depreciates to **$666**
* With this depreciation, max debt possible is **$466.20** \[$$666 * 0.70$$]
* This means that his collateral can no longer support the loan position of $500; eligible for liquidation
* This is reflected in the account health which falls to **< 1**\[$$\frac{666* 0.75}{500}$$].

{% hint style="info" %}
To see how health factor is calculated from a code perspective: [calculateUserAccountDataParams](../functions/common-functions/calculateuseraccountdataparams.md)
{% endhint %}

### **Close Factor**

The percentage of an unhealthy loan that can be repaid in a single liquidate transaction.

* Therefore **`maxLiquidatableDebt = userTotalDebt * closeFactor`**

If user's health factor > **0.95** \[`CLOSE_FACTOR_HF_THRESHOLD`]

* only a maximum of **50%** (`DEFAULT_LIQUIDATION_CLOSE_FACTOR`) of the debt can be liquidated per valid `liquidationCall()`

If user's health factor $$\leq$$**0.95** \[`CLOSE_FACTOR_HF_THRESHOLD`]

* then **100%** (`MAX_LIQUIDATION_CLOSE_FACTOR`) of the debt can be liquidated in single `liquidationCall()`

**Code Reference:**

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

* Liquidators can set `debtToCover` to `uint(-1)` and the protocol will proceed with the highest possible liquidation allowed by the close factor.
* Partial Liquidation: In the event that the `debtToCover` is less than maxLiquidatableDebt, the loan position will be liquidated partially, based on `debtToCover`&#x20;

### Liquidation Bonus/Penalty

Liquidators are incentivised to hunt bad loans as a means of keeping the protocol healthy. They prevent Aave from being saddled with bad debt.

To that end, successful liquidations award the liquidator with a bonus. For example, if a liquidator liquidiates 100% of a user's debt, the user does not just lose the equivalent amount of collateral in base value. The target user suffers an additional haircut of liquidation penalty, which goes to the liquidator.

If the liquidation bonus 5%, the user loses collateral worth 105% of his debt.

Example:

* User has $100 of bad debt in some asset A; eligible for 100% liquidation
* Liquidator repays user's loan in full.
* Asset A has a liquidation bonus of 5%.
* Liquidator takes from user:
  * $100 of collateral -> to cover the debt repaid to protocol, on behalf of
  * $5 of collateral -> liquidation bonus; liquidator's profit

{% hint style="info" %}
Essentially, liquidation bonus is the profit enjoyed by the liquidator for services rendered.

* Liquidation bonus varies from asset to asset: [https://docs.aave.com/risk/v/aave-v2/asset-risk/risk-parameters](https://docs.aave.com/risk/v/aave-v2/asset-risk/risk-parameters)
* Liquidation bonus is based on the collateral asset selected to be liquidated; not the debt asset.
{% endhint %}

### Liquidation Fee&#x20;

The liquidation fee directs a share of the liquidation penalty to a collector contract from the ecosystem treasury.

Essentially, this is a tax levied on the profit enjoyed by liquidators. To continue the earlier example:

* Liquidator enjoys bonus of $5
* Liquidation fee for the collateral asset that was taken as bonus is 1%
* Liquidator pays $0.05 to the treasury

{% hint style="info" %}
The liquidation fee is determined by the collateral asset that the liquidator decides to liquidate, to cover the bad debt.

* User has asset A,B,C as collateral supporting debt in asset D.
* Liquidator has to pair one of the collaterals off against the debt asset he is looking to liquidate.
* If he chooses to repay asset D and liquidating asset A in the process, the liquidation fee he is subject to is determined by the liquidation fee parameter as set on asset A in its reserveConfiguration.&#x20;
{% endhint %}

##
