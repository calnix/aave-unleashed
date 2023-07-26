# StableDebtToken

* [https://docs.aave.com/developers/v/2.0/the-core-protocol/debt-tokens](https://docs.aave.com/developers/v/2.0/the-core-protocol/debt-tokens)
* talk about minting, burning
* avg stable rate calc





## \_accrueToTreasury

```solidity
cumulatedStableInterest = currAvgStableBorrowRate CompoundInterest(stableLastUpdateTimestamp -> LastUpdateTimestamp)
prevTotalStableDebt = currPrincipalStableDebt * cumulatedStableInterest
```

Problem

* how is the model we see in DefaultReserveInterestRateStrategy connected to the issuance/burning of stableTokens seen in stableDebtToken
*



ReserveCache has&#x20;

* currPrincipalStableDebt
* currAvgStableBorrowRate
* currTotalStableDebt
* stableDebtLastUpdateTimestamp

These three are obtained via .**getSupplyData**() on the stableDebtToken contract:

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption><p><strong>getSupplyData</strong></p></figcaption></figure>

* currPrincipalStableDebt => `super.totalSupply()`&#x20;
* currTotalStableDebt => `_calcTotalSupply(avgRate)`
* currAvgStableBorrowRate => `_avgStableRate`
* stableDebtLastUpdateTimestamp => `_totalSupplyTimestamp`



**Then**

* nextAvgStableBorrowRate = currTotalStableDebt&#x20;
* nexTotalStableDebt = currAvgStableBorrowRate&#x20;

## Connect defaultInterestRateStrategy to stableDebtToken

{% tabs %}
{% tab title="First Tab" %}


defaultInterestRate

* dictates how currentStableBorrowRate increments based on Utilization

**How does currentStableBorrowRate connect with a user's stable rate?**

* stored in reserveData
* updated when .updateRates::calculateInterestRates

::WITHIN CACHE: reserveCache holds

* currPrincipalStableDebt
* currTotalStableDebt
* currAvgStableBorrowRate
* stableDebtLastUpdateTimestamp -----------------------------------> getSupplyData
* nextTotalStableDebt = currTotalStableDebt
* nextAvgStableBorrowRate = currAvgStableBorrowRate

THIS IS THE CONNECTION: on borrowing stable,

* currentStableRate = reserve.currentStableBorrowRate
* stableToken.mint(user, onbehalfOf, amount, currentStableRate)

### the mint fn uses currentStableRate as follows:

* update \_avgStableRate
{% endtab %}

{% tab title="Second Tab" %}


1. cache reserveCache holds
2. currPrincipalStableDebt
3. currTotalStableDebt
4. currAvgStableBorrowRate
5. stableDebtLastUpdateTimestamp -----------------------------------> getSupplyData
6. nextTotalStableDebt = currTotalStableDebt
7. nextAvgStableBorrowRate = currAvgStableBorrowRate
8. borrow stable on borrowing stable:
9. stableToken.mint(user, onbehalfOf, amount, reserve.currentStableBorrowRate)
10. rate = currentStableBorrowRate

// the mint fn uses currentStableBorrowRate as follows: // calculating user's nextStableRate currentStableRate = \_userState\[borrower].additionalData nextStableRate = \[currentStableRate \* (currentBalance + amountInRay\*rate)] / (currentBalance + amount) \_userState\[borrower].additionalData = nextStableRate

//update \_avgStableRate currrentAvgStableRate = \_avgStableRate = = currrentAvgStableRate \* (previousSupply + rate\*amount) / nextSupply

return currrentAvgStableRate == reserveCache.nextAvgStableBorrowRate

3. Update Rate

averageStableBorrowRate == reserveCache.nextAvgStableBorrowRate

* used to calc overall borrow rate to get liquidity rate

\================================================================================================ currAvgStableBorrowRate currPrincipalStableDebt

* derived from getSupplyData
{% endtab %}
{% endtabs %}



