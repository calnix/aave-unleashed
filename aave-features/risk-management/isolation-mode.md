# Isolation Mode

## TLDR

* Assets listed as isolated assets can only be used collateral to borrow stablecoins.
* There is a global debt ceiling for each isolated asset -> max borrow system-wide.
* By only allowing borrow against stablecoins, Aave is able to set a debt ceiling in USD.
* If other assets like Ether is allowed, their price volatility would make it difficult it set a consistent debt ceiling.

## &#x20;Isolation Mode explained

At times, assets are introduced into Aave in isolation mode. This can be because they are new, have a short track record or may even carry unknown risks.&#x20;

Listing a new asset in isolation mode serves as a restrictive cautious introduction. If the market conditions around the asset progresses positively, restrictions can then be lifted.&#x20;

{% hint style="info" %}
* Listing a new token, however popular, that is 2 months old may introduce a number of risks, including the possibility of a rug pull.&#x20;
* The idea is to limit exposure to excessive price volatility and potential insolvencies.
* Therefore, introducing new assets into Aave via isolation mode helps safeguard users and minimize risk.
{% endhint %}

**Isolation mode limits a specific asset’s use as collateral in three ways:**

1. Other assets cannot be used as collateral at the same time.
2. Only approved stablecoins can be borrowed against the isolated asset.
3. Total loans against an isolated asset cannot exceed a pre-defined debt ceiling.

{% hint style="info" %}
Borrowers using an isolated collateral can only borrow stablecoins that have been permitted by the Aave governance to be borrowable in isolation mode.
{% endhint %}

#### Example

Token 2, is a new asset that has been listed as **isolated.** It has a **debt ceiling of 10M.**

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption><p><a href="https://docs.aave.com/developers/getting-started/readme#isolation-mode">https://docs.aave.com/developers/getting-started/readme#isolation-mode</a></p></figcaption></figure>

1. Chad supplies Token 2 as collateral.&#x20;
2. He is only able to borrow USDT, DAI and USDC against this collateral.
3. The total loans taken by everyone using Token 2 as collateral, including Chad, cannot exceed 10M.

{% hint style="info" %}
Chad can still supply other assets to earn supply interest. However, they cannot be used as collateral.
{% endhint %}

#### Gauntlet Methodology

* [https://medium.com/gauntlet-networks/methodology-isolation-mode-3b8d67eee695](https://medium.com/gauntlet-networks/methodology-isolation-mode-3b8d67eee695)
