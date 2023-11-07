# Mint debt token

## Overview

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] getIsolationMode
* [x] validateBorrow
* [ ] mint debt token
* [ ] setBorrowing&#x20;
* [ ] update IsolationMode debt
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer underlying to user

<img src="../../.gitbook/assets/file.excalidraw (24).svg" alt="" class="gitbook-drawing">

* If stable interest was selected -> mint stable debt tokens&#x20;
* Else -> mint variable debt tokens

## mint stableDebtTokens

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
rate: reserve.currentStableBorrowRate
{% endhint %}

### \_calculateBalanceIncrease

* For a user, calculates the increase in stable debt, due to interest accrued from timestamp of their last update, to current time.
* Serves to update user's prior stable debt

{% hint style="info" %}
`balanceIncrease` reflects interest accrued from lastUpdate till current block.timestamp.&#x20;

Does not relate to incoming borrow action.
{% endhint %}

### totalSupply

```solidity
    vars.previousSupply = totalSupply();
    vars.currentAvgStableRate = _avgStableRate;
    vars.nextSupply = _totalSupply = vars.previousSupply + amount;
```

#### vars.previousSupply = totalSupply();

`totalSupply` returns the updated supply value, by accounting for recently accrued interest, from `_totalSupplyTimestamp` till now, based on the `_avgStableRate`.

* `_avgStableRate`: internal storage variable on StableDebtToken.sol
* `previousSupply`: updated supply value

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

* `principalSupply` => `super.totalSupply` calls `totalSupply` on `IncentivizedERC20` which returns `_totalSupply`

{% hint style="info" %}
`previousSupply` is set to `totalSupply()` instead of `_totalSupply` directly, so that the updated supply value can be obtained.&#x20;
{% endhint %}

With recent interest accounted for, the storage variable `_totalSupply` is updated with the incoming stable debt, `amount`.&#x20;

```solidity
vars.nextSupply = _totalSupply = 
    vars.previousSupply + amount;
```

### Calculate new stable rate

* `nextStableRate` is new stable rate, specific to the user

<figure><img src="../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```solidity
nextStableRate = 
(_avgStableRate * currentBalance) + (amount * rate / currentBalance + amount)
```
{% endcode %}

* `nextStableRate` stored in the user's `UserState.additionalData` struct. `userState` mapping collects all users' `UserState` structs.
* Store update time in `_timestamps`.`_timestamps` mapping collects the last update timestamp for all users.
* Update the timestamp of the last update of  `_totalSupply.`

{% hint style="info" %}
rate: `reserve.currentStableBorrowRate`
{% endhint %}

### Update average Stable rate

* `_avgStableRate`: internal uint128 on StableDebtToken.sol
* `vars.currentAvgStableRate` was set to `_avgStableRate` at the start of `mint`

<figure><img src="../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```solidity
newAvgStableRate = 
(_avgStableRate * previousSupply) + [(rate * amountInRay) / nextSupply]
```
{% endcode %}

{% hint style="info" %}
* previousSupply: updated`totalSupply`
{% endhint %}

### mint tokens

<figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

Notice that we do not just mint as per the incoming stable debt amount; but also mint for `balanceIncrease`. This serves to update user's prior stable debt based on interest accrued over time elapsed since last update.

* Since we going to mint, mint to update the user's entire position

`mint` calls the internal function `_mint`:

<figure><img src="../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

## mint variableDebtTokens

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

* `amountScaled`: scale the borrow amount against variableBorrowIndex
* See variableDebtToken [mint](../../contracts/variabledebttoken.md#mint)
