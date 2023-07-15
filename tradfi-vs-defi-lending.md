# TradFi vs DeFi: Lending

In the tradfi world, lending and borrowing are the cornerstone of economic growth and financial stability. Banks and financial institutions have facilitated these essential activities, providing individuals and businesses with access to capital and liquidity.&#x20;

#### Banking: Lenders x Borrowers

* Lenders loan their capital to borrowers with the objective of profiting via an interest premium. Banks act as intermediaries, facilitating this process and assessing the creditworthiness of borrowers.&#x20;
* Borrowers look to leverage these funds to finance projects, expand businesses, or meet personal financial needs.
* Interest rates are influenced by factors such as market conditions, central bank policies, and other factors pertaining to macroeconomic and risk outlook.

#### On-chain lending

Parallel to the traditional lending model, on-chain lending and borrowing protocols like Aave enable users to either

* loan their assets to borrowers,
* borrow assets by providing collateral.&#x20;

The lending process is facilitated by smart contracts, **removing the need for institutional intermediaries and enabling a peer-to-peer or peer-to-protocol approach.**

* Lenders supply liquidity to the protocol, which powered by smart contracts, aggregates liquidity.
* Borrowers can access the aggregated liquidity to take out loans accordingly.&#x20;
* Interest rates are determined algorithmically, influenced by factors such as supply and demand dynamics, utilization rates, and protocol-specific mechanisms.

{% hint style="info" %}
* While tradfi interest rates are influenced through money market operations and other policies, defi interest rates are determined by on-chain interest rate models.
* The parameters of these models are periodically updated to achieve higher capital efficiency in-line with market conditions
* See Gauntlet's proposals on Aave: [example](https://governance.aave.com/t/arfc-aave-v3-interest-rate-curve-recommendations-from-gauntlet-2023-04-27/12921)
{% endhint %}

### Collateralization and Credit

Another crucial distinction between Tradfi and Defi lending is that of collateralization.&#x20;

**Tradfi operates on credit**

* Credit can generally be broken down into secured and unsecured credit.&#x20;
* A mortgage is a type of secured credit, as the loan is collateralized by the property.&#x20;
* A credit card is a type of unsecured credit, or one can regard it as being collateralized by the creditworthiness of the borrower.

{% hint style="info" %}
Tradfi allows for undercollateralized lending. If you had 1 million and decent credit worthiness, you could borrow 10 million from the bank.
{% endhint %}

**Defi operates on overcollateralization**

* Majority of existing lending protocols operate on an overcollateralized basis.
* Borrowers must lock up a portion of their crypto assets as collateral, which serves as a safeguard for lenders in case of default.&#x20;
* If you offer 1 million in collateral, you could borrow up to 750K.

This approach enhances the security and stability of the lending process, providing lenders with reassurance of solvency of the protocol with which they have opted to loan their assets.&#x20;

While this might seem to be a negative, on-chain lending protocols offer a decentralized, transparent, and accessible alternative to traditional financial systems.&#x20;

They enable individuals from all corners of the world to participate in lending and borrowing activities, without the limitations and complexities associated with traditional banking institutions.

{% hint style="success" %}
No hassle of KYC, AML or credit assessments. As long you have a wallet and some on-chain assets you are free to participate in lending and borrowing.&#x20;
{% endhint %}

{% hint style="info" %}
Managing an on-chain debt position is very similar to managing a leveraged trading position due the requirements of overcollateralization and liquidation. We will talking explain more in the Liquidation section.
{% endhint %}
