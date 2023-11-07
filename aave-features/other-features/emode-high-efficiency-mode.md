---
description: https://docs.aave.com/developers/whats-new/efficiency-mode-emode
---

# eMode: High efficiency Mode

## TLDR

* If both collateral and borrowing assets are in the same eMode category, the borrower is allowed a higher collateralization factor.
* This is applicable only in cases where the collateral and borrowing assets have low price volatility against each other.
* This makes it safer to allow users to borrow more.

## High efficiency mode

The High Efficiency Mode or _**eMode**_ allows borrowers to extract the highest borrowing power out of their collateral when collateral and borrowed assets are correlated in price, particularly when both are derivatives of the same underlying asset (eg. stablecoins pegged to USD).

* The E-mode feature maximizes capital efficiency when collateral and borrowed assets have correlated prices.&#x20;
* For example, DAI, USDC, USDT are all stablecoins pegged to USD. These stablecoins are all within the same E-mode category.&#x20;
* Accordingly, a user supplying DAI in E-mode will have higher collateralization power when borrowing assets like USDC or USDT.&#x20;
* Only assets of the same category (for example stablecoins) can be borrowed in E-mode.
* E-mode does not restrict the usage of other assets as collateral. Assets outside of the E-mode category can still be supplied as collateral with normal LTV and liquidation parameters.

<figure><img src="../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

[https://docs.aave.com/developers/whats-new/efficiency-mode-emode](https://docs.aave.com/developers/whats-new/efficiency-mode-emode)&#x20;

* If the user has supplied liquidity to the protocol, the user eMode category is set to 0 by default.
* If user is not borrowing anything, can activate eMode
* If user is already borrowing an asset that is NOT in his eMode category, cannot activate eMode.

### How do I enter E-mode?

* To enter E-mode from the dashboard, toggle to the dropdown menu under the “"Your Borrows”" where you will find an “Enable E-mode” button.&#x20;
* Initially, the button to enable E-mode will indicate that E-mode is “disabled.” Click “Enable E-mode" ” and follow the instructions in the pop-up.&#x20;
* Once you have followed the instructions, you will have enhanced borrowing power (i.e., up to 97% LTV) within E-mode and can only borrow assets within the same category of assets (for example, stablecoins).





{% hint style="info" %}
This can enabling a wave of new use cases such as High leverage forex trading, Highly efficient yield farming (for example, deposit ETH staking derivatives to borrow ETH), Diversified risk management etc.
{% endhint %}

### Aave example

[https://governance.aave.com/t/introducing-aave-v3/6035](https://governance.aave.com/t/introducing-aave-v3/6035)

<figure><img src="../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

An example:

```
Protocol defines eMode category 1 (stablecoins) as follows: 

97% LTV 

98% Liquidation threshold 

2% Liquidation bonus 

No custom price oracle 

Karen chooses eMode Category 1 (stablecoins) 

Karen supplies DAI (which normally has 75% LTV) 

Karen can borrow other stablecoins in Category 1 (including DAI) with the borrowing power defined by the eMode category (97%).  Karen’s capital is therefore 22% more efficient.  
```

Note: In this example, Karen can supply other non-Category 1 assets to use as collateral; however, only assets belonging to the same eMode category chosen by the user will have enhanced category-specific risk parameters.

V3 eMode can support up to 255 categories.
