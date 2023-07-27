---
description: Borrow Demand -> Utilization -> Borrow rate -> Supply rate
---

# Market forces x Interest Rate Models

## Summary

In a nutshell, market demand for loans (borrow demand) determines the level of utilization (U), which in turn determines the borrow interest rate. The borrow interest rate discounted by a factor is the supply interest rate.&#x20;

$$
Borrow Demand => Utilization => InterestRate_{borrow} => InterestRate_{supply}
$$

The relationship between utilization and borrow rates is defined by the lending protocol via their interest rate model. The model dictates how borrow interest rate would increase with utilization levels, for a specific asset.&#x20;

Typically, the interest model is kinked, comprising of two different gradients; one for low levels of utilization and another for high levels of utilization. This allows for a more efficient pricing of the cost of borrowing against fluctuating market demand for loans.

#### **Borrowers and Suppliers**

The crux of relationship between lenders and borrowers is this:

$$
Borrow Interest = Supply Interest + Premium
$$

The interest accrued by borrowers is the interest accrued by suppliers, with a premium on the top. The premium essentially is the cost of using the service. This margin could go to the protocol as revenue, accumulated in the treasury or a number of other things, depending on the protocol.&#x20;

{% hint style="info" %}
In the case of Aave, the premium goes to the Aave Ecosystem Reserve.
{% endhint %}

## Determining interest rates

Aave's interest rates are determined algorithmically to strike a balance between utilization and protocol solvency. This approach allows the protocol to dynamically adjust interest rates based on supply and demand dynamics, ensuring efficient allocation of liquidity and maintaining the financial stability of the protocol.

This brings us to the concept of Utilization.&#x20;

### Utilization

Utilization (U) is the percentage of a specific asset that is lent out with respect to its total supply.

$$
U =  \frac{Borrowed }{Total \, Deposits} =   \frac{Borrowed}{Borrowed + Free\, Deposits } \\
$$

The level of utilization is a reflection of market demand for borrowing. Utilization in turn determines interest rates.

**When the utilization rate is high:** (indicative of high demand for borrowing)

* Borrow and supply interest rates rises&#x20;
* Incentivize lenders to supply more assets, disincentivize wanton borrowing&#x20;
* Consequently, increasing the available liquidity within the protocol and tempers borrowing demand, leading to an equilibrium point.&#x20;

**On the other hand, if the utilization rate is low:** (indicating excess idle liquidity)

* Borrow and supply rates fall
* Incentivizes borrowing and disincentivizes lending&#x20;
* Consequently, reducing idle liquidity and maximizing the utilization of assets within the protocol.&#x20;

{% hint style="info" %}
This dynamic adjustment mechanism ensures that interest rates are reflective of the real-time supply and demand conditions, fostering efficient capital allocation.
{% endhint %}

Protocol solvency is another critical consideration in determining interest rates algorithmically. If interest rates were solely determined by market forces without considering the solvency aspect, there could be a risk of instability or insolvency of the protocol. Therefore, the interest rate mechanism aims to maintain a delicate balance between competitiveness in the market and the financial health of the protocol.

{% hint style="info" %}
Aave v3 complements the interest rate mechanism with borrow caps to manage insolvency risk.
{% endhint %}

### Borrow Interest Rates

* Borrow interest rates are a function of Utilization
* The relationship could be linear or a piecewise function (kinked)
* Parameters in the function can be set by protocol governance

<figure><img src=".gitbook/assets/image (187).png" alt=""><figcaption><p>Aave: Dai Interest rate model</p></figcaption></figure>

The interest rate function for DAI is a piecewise function with an optimal utilization rate. Beyond this optimal point, the interest rate increases at a faster rate.&#x20;

#### Piecewise function

The piecewise function is defined as such:

$$
R_{borrow} = \begin{cases}
    R_{intercept} \,+ \frac{U}{U_{optimal}}R_{slope1}, & \text{if } U \leq U_{optimal} \\
    R_{intercept} \,+ R_{slope1} + \frac{U - U_{optimal}}{1-U_{optimal}}R_{slope2},  & \text{if } U > U_{optimal}
\end{cases}
$$

The total borrow rate is composed of the rate at zero utilization ( $$R_{intercept}$$) plus the one or both utilization slopes. If you think about it, the piecewise function is basically two lines following: $$y = mc + c$$.

#### For   $$U \leq U_{optimal}$$ &#x20;

<figure><img src=".gitbook/assets/image (142).png" alt="" width="563"><figcaption></figcaption></figure>

#### For   $$U > U_{optimal}$$ &#x20;

<figure><img src=".gitbook/assets/image (229).png" alt="" width="563"><figcaption></figcaption></figure>

**Why piecewise, instead of a single linear function?**

When utilization exceeds optimal levels, concerns of insolvency become more significant. Hence, the variable borrow interest rate increases at a much higher rate - represented by the steeper gradient.&#x20;

This would serve to more heavily incentivize suppliers to lend, thereby increasing liquidity, while simultaneously tempering borrowing to restore healthy idle liquidity levels and avoid insolvency.&#x20;

### Combined

Here is the combined graph for the piecewise function, using DAI as an example.

<figure><img src=".gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

The parameters are as follows:

```python
R_intercept = 0
R_slope1 = 0.04
R_slope2 = 0.75
U_optimal = 0.80
```

These parameters were obtained from DAI's `DefaultReserveInterestRateStrategy` contract, and are accurate at time of writing.&#x20;

#### Parameters

Each asset has its interest rate parameters stored on-chain via a `DefaultReserveInterestRateStrategy` contract.&#x20;

* DAI: [https://etherscan.io/address/0x694d4cFdaeE639239df949b6E24Ff8576A00d1f2](https://etherscan.io/address/0x694d4cFdaeE639239df949b6E24Ff8576A00d1f2#readContract)

<figure><img src=".gitbook/assets/image (1) (2) (1).png" alt=""><figcaption><p>on-chain parameters</p></figcaption></figure>

From time to time, parameters are modified to better suit market conditions. Proposals to update these parameters are submitted and passed via Aave's governance.

{% hint style="info" %}
See Gauntlet's proposals on Aave: [example](https://governance.aave.com/t/arfc-aave-v3-interest-rate-curve-recommendations-from-gauntlet-2023-04-27/12921)
{% endhint %}

### Supply Interest Rate

* Supply interest rates are dependent on borrow rates.&#x20;
* The interest rate provided to depositors is determined by multiplying the borrow rate by the utilization rate.
* This ensures that the interest earned from borrowers matches the interest paid to depositors, maintaining a balance in the amounts exchanged between the two parties.

{% hint style="info" %}
Supply interest rate is also referred to as the liquidity rate.
{% endhint %}

<figure><img src=".gitbook/assets/image (129).png" alt="" width="563"><figcaption><p>change picture with your notes</p></figcaption></figure>

**Example:** assume a lending protocol where the borrow rate is 10% and the utilization rate is 50%. In this case, the interest rate offered to depositors would be:

* Interest Rate = Borrow Rate \* Utilization Rate&#x20;
* Interest Rate = 10% \* 50%&#x20;
* Interest Rate = 5%

Depositors would earn an interest rate of 5% on their deposits. The total interest payment received by depositors would be equivalent to the total interest paid out by borrowers.&#x20;

While this simple example is useful for our understanding, it ignores the margin we mentioned at the start; the cut that goes to the protocol, which could reflect fees, treasury contribution, etc.

In Aave, supply interest rate is calculated as such:

```solidity
liquidityRate = overallBorrowRate * supplyUsageRatio * (1 - reserveFactor)
```

In the later stages, we will get a higher resolution understanding of what `overallBorrowRate` and the other terms mean, but for now we can understand this as follows:

```
liquidityRate = borrowRate * UtilizationRate  * (1 - reserveFactor)
```

Notice the introduction of the reserve factor - it determines the split between what the depositors enjoy versus what flows into Aave's treasury.&#x20;

{% hint style="info" %}
The reserve factor is a percentage of protocol interest which goes to the Aave Ecosystem Reserve.
{% endhint %}
