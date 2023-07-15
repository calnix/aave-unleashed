---
description: https://docs.aave.com/risk/asset-risk/risk-parameters#liquidation-threshold
---

# Liquidation

## Overview

Liquidation on Aave is a mechanism designed to protect lenders and incentivize repayment of outstanding loans.&#x20;

* Borrowers must maintain a **minimum health factor**, which compares their collateral value to their outstanding loan value.&#x20;
* If a borrower's loan position falls below the minimum health factor, their position becomes eligible for liquidation.
* During the liquidation, the borrower's collateral is seized and sold in the open market to repay the outstanding loan.
* A liquidation penalty is applied to discourage borrowers from allowing their position to become unhealthy.
* Liquidators are offered incentives to liquidate unhealthy positions; this is financed by the liquidation penalty.&#x20;

The liquidation mechanism aims to protect lenders from the risk of default and maintain the stability and security of the lending platform. It incentivizes borrowers to maintain sufficient collateralization and encourages active participation from liquidators who can profit from liquidating unhealthy positions.

### Loan To Value: LTV

_**maybe move LTV to under borrowing**_

* Defines the maximum amount of assets that can be borrowed with a specific collateral.
* Expressed as a percentage
* For example, ETH has an LTV of 75% -> for every 1 ETH worth of collateral, borrowers will be able to borrow 0.75 ETH worth of the corresponding asset

{% hint style="warning" %}
LTV is always < 100%. Recall that Aave operates on overcollateralized lending.
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
* For example, a Liquidation threshold of 80% means that if the loan value rises above 80% of the collateral value, the position is undercollateralized and could be liquidated.
* The delta between the LTV and the Liquidation Threshold is a safety mechanism in place for borrowers.

{% hint style="info" %}
For example LTV at 70% and a liquidation threshold of 75% means that users have a 5% margin of safety.

* can take on a max debt position valued at 70% of collateral
* will be liquidated if debt position is valued at 75% of collateral
* 5% safety margin to endure price volatility&#x20;
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



## Liquidation&#x20;

### **Close Factor**

* The percent, ranging from 0% to 100%, of a liquidatable account's borrow that can be repaid in a single liquidate transaction.
* Therefore **`maxLiquidatableDebt = userTotalDebt * closeFactor`**

If user's health factor > **0.95** \[`CLOSE_FACTOR_HF_THRESHOLD`]

* only a maximum of **50%** (`DEFAULT_LIQUIDATION_CLOSE_FACTOR`) of the debt can be liquidated per valid `liquidationCall()`

If user's health factor $$\leq$$**0.95** \[`CLOSE_FACTOR_HF_THRESHOLD`]

* then **100%** (`MAX_LIQUIDATION_CLOSE_FACTOR`) of the debt can be liquidated in single `liquidationCall()`

**Code Reference:**

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

* Liquidators can set `debtToCover` to `uint(-1)` and the protocol will proceed with the highest possible liquidation allowed by the close factor.
* Partial Liquidation: In the event that the `debtToCover` is less than maxLiquidatableDebt, the loan position will be liquidated partially, based on `debtToCover`&#x20;

### Liquidation Incentive



### Liquidation penalty

The liquidation penalty is a fee rendered on the price of assets of the collateral when liquidators purchase it as part of the liquidation of a loan that has passed the liquidation threshold.



### Liquidation Factor&#x20;

The liquidation factor directs a share of the liquidation penalty to a collector contract from the ecosystem treasury.

## Example

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

* [https://docs.google.com/spreadsheets/d/1DRLIDCo-tUKF13NJoo9od1FmvUnrjeHibKzhZJ48Ar0/edit#gid=550821971](https://docs.google.com/spreadsheets/d/1DRLIDCo-tUKF13NJoo9od1FmvUnrjeHibKzhZJ48Ar0/edit#gid=550821971)
* modify the example with the correct close factor, liquidation incentive, penalties, etc.&#x20;

### Here's an overview of how liquidation works on Aave:

1. Collateralization Ratio: Each borrower on Aave must maintain a minimum collateralization ratio for the safety of lenders. The collateralization ratio is the value of the borrower's collateral compared to the value of their outstanding loan. If the ratio falls below a certain threshold, the borrower's position becomes vulnerable to liquidation.
2. Health Factor: Aave calculates a health factor for each borrower, which is the ratio of the borrower's collateral value to their outstanding loan amount, adjusted for risk parameters. If the health factor falls below a predefined threshold, typically 1, the position is considered unhealthy and subject to liquidation.
3. Liquidation Incentives: To encourage other users to liquidate unhealthy positions, Aave offers liquidators an incentive in the form of a discount on the outstanding loan. Liquidators can purchase the borrower's debt at a discounted price, allowing them to make a profit when repaying the loan using the borrower's collateral.
4. Flash Loan Mechanism: Aave's liquidation process utilizes the flash loan mechanism, which allows users to temporarily borrow funds without providing collateral. Liquidators can borrow the required amount to repay the outstanding loan and initiate the liquidation process.
5. Collateral Seizure: During the liquidation, the borrower's collateral is seized and sold in the open market to repay the outstanding loan. The liquidation process ensures lenders are repaid and incentivizes borrowers to maintain a healthy collateralization ratio.
6. Liquidation Penalty: In addition to the discount received by the liquidator, a liquidation penalty is applied to the borrower. The penalty is a fee charged to the borrower to discourage them from allowing their position to become unhealthy. The penalty amount is typically a percentage of the outstanding loan.



{% hint style="info" %}
Liquidators must already have sufficient balance of the debt asset, which will be used by the `liquidationCall` to pay back the debt. They can use `flashLoan` for liquidations.
{% endhint %}



**Links**

* [https://docs.aave.com/faq/borrowing#when-could-my-stable-rate-be-rebalanced](https://docs.aave.com/faq/borrowing#when-could-my-stable-rate-be-rebalanced)
* [https://mymerlin.io/website/liquidation-on-aave/](https://mymerlin.io/website/liquidation-on-aave/)
* [https://docs.aave.com/risk/asset-risk/risk-parameters](https://docs.aave.com/risk/asset-risk/risk-parameters)
* [https://docs.aave.com/faq/liquidations](https://docs.aave.com/faq/liquidations)
* [https://docs.aave.com/developers/guides/liquidations](https://docs.aave.com/developers/guides/liquidations)





#### Compeititve liquidaiton

* how/.why competitive
* compare to otehr mechanisms
