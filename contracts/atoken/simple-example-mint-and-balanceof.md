# Simple example: mint & balanceOf

### `AToken::balanceOf` returns:

```solidity
super.balanceOf(userAddress) * POOL.getReserveNormalisedIncome(_underlyingAsset)
```

### AToken employs a unit increase model

* deposit 1 DAI -> user receives 1 aDAI
* interest is accumulated over time
* user's aDAI balance increases, reflecting incrementing interest
  * 1 aDAI -> 1.5 aDAI
* user redeems aTokens 1:1 with underlying
  * can redeem 1.5 DAI

### On deposit/supply:

User's deposit is recorded as "scaledBalance":

$$scaledBalance = \frac{amountDeposited}{currentLiquidityIndex}$$

scaledBalance is the actual value stored on-chain, within `_userState[userAddress].balance`.

{% hint style="info" %}
All deposits are scaled against the liqudity index and stored as scaled values via `_userState[userAddress].balance`
{% endhint %}

**Example**

<figure><img src="../../.gitbook/assets/image (247).png" alt=""><figcaption></figcaption></figure>

### mint

The user is minted aTokens according to the scaledBalance, not the deposit value.&#x20;

* however, balanceOf is calculated by scaling up the scaledBalance against the incrementing liquidity index.
* this is why, from the end-user's perspective it looks like they have minted aToken 1:1 to their deposits, and
* their aToken balance continually increments over time - because the liquidity index increments

### AToken balance

```solidity
aTokenBalance = scaledBalance * currLiquidityIndex
```

* LiquidityIndex only increments&#x20;
* An account's aTokenBalance can never go down, unless a withdrawal occurs

**Simplified illustration:**

```solidity
super.balanceOf(userAddress) * POOL.getReserveNormalisedIncome(_underlyingAsset)

_userState[userAddress].balance * reserve.LiquidityIndex

scaledBalance * currLiquidityIndex
```



**Links**

* [https://medium.com/coinmonks/how-aave-hacked-the-erc-20-token-in-the-most-beautiful-way-3e04fb4410fb](https://medium.com/coinmonks/how-aave-hacked-the-erc-20-token-in-the-most-beautiful-way-3e04fb4410fb)
* [https://mirror.xyz/freesuton.eth/EagGvVQZtLhsnnsTTwhHA6VwekMbImMmvniSWAm6SNg](https://mirror.xyz/freesuton.eth/EagGvVQZtLhsnnsTTwhHA6VwekMbImMmvniSWAm6SNg)
