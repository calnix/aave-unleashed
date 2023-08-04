---
description: >-
  https://docs.aave.com/developers/deployed-contracts/v3-mainnet/ethereum-mainnet
---

# DefaultReserveInterestRateStrategy

The DefaultReserveInterestRateStrategy contract is essentially the on-chain interest rate model. It contains two key functions:

1. `calculateInterestRates`
2. `_getOverallBorrowRate`

`calculateInterestRates` calculates the latest supply, variable borrow and stable borrow rates as explained previously.

`_getOverallBorrowRate` calculates the overall borrow rate given how much stable and variable rate loans have been taken.

### Parameters

Each asset has its interest rate parameters stored on-chain via a `DefaultReserveInterestRateStrategy` contract.&#x20;

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

These parameters are set in the constructor on deployment. We can explore their values by interacting with the deployed contract. For example, DAI contract on mainnet:&#x20;

[https://etherscan.io/address/0x694d4cFdaeE639239df949b6E24Ff8576A00d1f2#readContract](https://etherscan.io/address/0x694d4cFdaeE639239df949b6E24Ff8576A00d1f2#readContract)&#x20;

<figure><img src="../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption><p>on-chain parameters</p></figcaption></figure>

```python
R_intercept = 0
R_slope1 = 0.04
R_slope2 = 0.75
U_optimal = 0.80
```

These parameters were obtained from DAI's `DefaultReserveInterestRateStrategy` contract, and are accurate at time of writing.&#x20;

{% hint style="info" %}
Other contract addresses: [https://docs.aave.com/developers/deployed-contracts/v3-mainnet/ethereum-mainnet](https://docs.aave.com/developers/deployed-contracts/v3-mainnet/ethereum-mainnet)
{% endhint %}

{% hint style="success" %}
Periodically, parameters are modified to better suit market conditions. Proposals to update these parameters are submitted and passed via Aave's governance.
{% endhint %}

## \_getOverallBorrowRate

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Calculates the overall borrow rate as the weighted average between the total variable debt and total stable debt.

## calculateInterestRates

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Executed as part of **.updateInterestRates**, it is usually ran towards the end of a state-changing function (borrow, supply, etc). It calculates the new interest rates for

* supply
* variable borrow
* stable borrow

given that liquidity conditions and utilization have changed. This is well explained in the common functions section: [calculateInterestRates](../functions/common-functions/.updateinterestrates.md#.calculateinterestrates).
