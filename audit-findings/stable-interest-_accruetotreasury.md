# stable interest \_accrueToTreasury

## TLDR

previousTotalStableDebt is ignored when accounting for what is owed to the treasury

## \_accrueToTreasury

The purpose of `_accrueToTreasury` is to account for unbooked interest from the last update til now - and increment what is owed to the treasury accordingly.

* **Unbooked interest**: interest accrued from `reserve.`**`lastUpdateTimestamp`** till **now**&#x20;
* `reserveCache.reserveLastUpdateTimestamp` == `reserve.lastUpdateTimestamp` , as assigned in `.cache`

Calculating the unbooked interest for variable borrows is easy enough. It is the difference between nextVariableBorrowIndex and currVariableBorrowIndex.

{% hint style="info" %}
Objective: obtain accrued interest from **reserveCache.`reserveLastUpdateTimestamp` to now**
{% endhint %}

**For stable borrows, this will be different as there is no stable borrow index.**

Let's extract the parts relevant for stable borrows:

```solidity
//calculated in .cache:
reserveCache.currTotalStableDebt = _calcTotalSupply(avgRate)

//...
//within _accrueToTreasury:

//calculate the stable debt until the last timestamp update
vars.cumulatedStableInterest = MathUtils.calculateCompoundedInterest(
  reserveCache.currAvgStableBorrowRate,
  reserveCache.stableDebtLastUpdateTimestamp,
  reserveCache.reserveLastUpdateTimestamp
);

//prevTotalStableDebt = currPrincipalStableDebt * cumulatedStableInterest
vars.prevTotalStableDebt = 
reserveCache.currPrincipalStableDebt.rayMul(vars.cumulatedStableInterest);
```

**Therefore:**

**`currTotalStableDebt`** - **`prevTotalStableDebt`** = interest accrued from `reserveLastUpdateTimestamp` till now.

{% hint style="info" %}
variable interest is similarly calculated from reserveCache.`reserveLastUpdateTimestamp`to now
{% endhint %}

* **`currTotalStableDebt:`** accounts for interest from `_totalSupplyTimestamp` till now
* **`prevTotalStableDebt:`** accounts for interest from `stableDebtLastUpdateTimestamp` to `reserveLastUpdateTimestamp`

{% hint style="info" %}
`stableDebtLastUpdateTimestamp == _totalSupplyTimestamp`

* assigned via `getSupplyData` in `.cache`
{% endhint %}

<figure><img src="../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

**Why can't we simply calculate interest from `reserveLastUpdateTimestamp` like variable?**

* **`reserveLastUpdateTimestamp` >= `_totalSupplyTimestamp`**

**`_totalSupplyTimestamp`** is only updated when mint/burn of the stableDebtToken are called. However, **`reserveLastUpdateTimestamp`** is updated in `.updateState`, which is called on every state-changing function.

Therefore, **`_totalSupplyTimestamp`** is likely to be an older value.&#x20;





**On time**

`_totalSupplyTimestamp`&#x20;

* updated during mint/burn

`stableDebtLastUpdateTimestamp`&#x20;

* loaded into reserveCache via getSupplyData
* getSupplyData passes `_totalSupplyTimestamp` into `stableDebtLastUpdateTimestamp`&#x20;

{% hint style="info" %}
**`stableDebtLastUpdateTimestamp`** **`==`** **`_totalSupplyTimestamp`**&#x20;
{% endhint %}



Order

* .cache&#x20;
  * loads **`_totalSupplyTimestamp`  as** reserveCache.**stableDebtLastUpdateTimestamp**
  * ```solidity
    // accounts interest from _totalSupplyTimestamp till now
    reserveCache.currTotalStableDebt = _calcTotalSupply(avgRate)
    ```
* .updateState
  * \_accrueToTreasury
  * ```solidity
    //accounts interest
    //from stableDebtLastUpdateTimestamp to reserveLastUpdateTimestamp
    vars.prevTotalStableDebt = 
    reserveCache.currPrincipalStableDebt.rayMul(vars.cumulatedStableInterest);
    ```

###
