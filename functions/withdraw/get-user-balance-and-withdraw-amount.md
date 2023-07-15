# get user balance & withdraw amount

## Overview

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [ ] get user's AToken balance + amount to withdraw
* [ ] validateWithdraw: confirm user has sufficient balance and asset isActive
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] collateral check
* [ ] burn ATokens
* [ ] Ensure existing loans are not impacted&#x20;

## get user balance

Check user's balance of ATokens corresponding to the asset their wish to withdraw. His aToken balance will dictate how much of the underlying is made available for withdrawal.

{% code overflow="wrap" fullWidth="true" %}
```solidity
uint256 userBalance = 
IAToken(reserveCache.aTokenAddress).scaledBalanceOf(msg.sender).rayMul(reserveCache.nextLiquidityIndex);
```
{% endcode %}

* `scaledBalanceOf` returns the user's aToken balance scaled against the liquidity index at the time of supply.
  * $$scaledBalanceOf = Deposit/liquidtyIndex$$

This scaled balance is multiplied against the current liquidity index, which was updated in `updateState`.

{% hint style="success" %}
For in-depth explanation of this behaviour, see: [AToken-balanceOf](../../scaling-and-atokens.md#atoken-balanceof)
{% endhint %}

{% hint style="info" %}
Supplied assets accrue supply interest, as accounted for by the scaling against current liquidity index
{% endhint %}

## get withdraw amount

```solidity
uint256 amountToWithdraw = params.amount;

if (params.amount == type(uint256).max) {
    amountToWithdraw = userBalance;
}
```

* If the supplied input is the maximum uint256 value, the withdrawal amount is set to the user's balance of ATokens => withdraw everything
* Otherwise, withdraw the amount as dictated by user input.
