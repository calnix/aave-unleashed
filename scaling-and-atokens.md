# Scaling and ATokens

Previously we explained how deposits were scaled against the liquidity index to reflect every user's unique starting point in the timeseries of interest accumulation.

That brings us to the topic of ATokens and the question of how they are minted - on the basis of the scaled value of deposit or otherwise.

## ATokens

Atokens are interest-bearing tokens that represent a user's share in the underlying deposited assets. When users deposit funds into the Aave protocol, they receive Atokens in return, which represent their entitlement to a portion of the pool's reserves.

{% hint style="info" %}
* User deposits 100 DAI, receives 100 aDAI in return. &#x20;
* ATokens are redeemable 1:1 for their underlying.
{% endhint %}

A depositor's balance of aTokens will increment over time as interest is accrued.&#x20;

**Example:**

* You deposit 100 DAI and are minted 100 aDAI token.&#x20;
* Over time, your aDAI balance will increment beyond 100, reflecting interest accrued.
* When you look to withdraw in full, you will receive more than 100 DAI.

Therefore, the growth of aToken balance reflects the interest earned on deposits over a period.

{% hint style="info" %}
You can see your AToken balance increase in real-time directly in your wallet.
{% endhint %}

Now we run into a contradiction. In the earlier [example](on-indexes/#scaling-positions) on scaling positions, we saw that a user who deposited 1000 DAI when the liquidity index was 1.1, had his deposit scaled to 909.

Does this mean that he is minted 909 aDAI? Yes and no:

* user will see that he has 1000 aDAI in his wallet
* however, the amount parameter passed into `mint` will be the `scaledbalance` of 909

To make more sense of this, let's look at how `balanceOf` works in the context of aTokens.

### AToken: `balanceOf`

Aave implements a modified version of the ERC-20 standard for its aTokens, such that the balance of aTokens increments without any transactions.&#x20;

Let's examine the implementation for `balanceOf`:

<figure><img src=".gitbook/assets/image (180).png" alt=""><figcaption><p><a href="https://github.com/aave/aave-v3-core/blob/29ff9b9f89af7cd8255231bc5faf26c3ce0fb7ce/contracts/protocol/tokenization/AToken.sol#L128">AToken::balanceOf</a></p></figcaption></figure>

Notice the use of override - it serves to override `balanceOf` functions declared within its inheritance tree, so that the parent functions do not get called.&#x20;

A user's aToken balance is the multiplication of two components:

1. `super.balanceOf`&#x20;
2. `POOL.getReserveNormalizedIncome()`

In a more digestible form, this basically translates to:

```solidity
super.balanceOf(user) * POOL.getReserveNormalizedIncome(_underlyingAsset)
        scaledBalance * currentLiquidityIndex
```

**Execution flow:**&#x20;

<figure><img src=".gitbook/assets/image (318).png" alt=""><figcaption><p>super.balanceOf(user)</p></figcaption></figure>

#### `super.BalanceOf`&#x20;

* Each user has a struct, `UserState` associated with their address via the mapping `_userState`
* The `balance` element within the `UserState` stores user's scaled balance&#x20;
* `super.balanceOf` returns `_userState[account].balance`

`super.BalanceOf` returns the scaled balance for a user.

#### `POOL.getReserveNormalizedIncome()`

<figure><img src=".gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

You might wonder what `calculateLinearInterest` does, we will explain this in a later section. For now you can operate on the understanding that `getReserveNormalizedIncome` returns the latest liquidity index, referred to as `currentLiquidityIndex`.&#x20;

So in conclusion,

```solidity
       super.balanceOf(user) * POOL.getReserveNormalizedIncome(_underlyingAsset)
 _userState[account].balance * currentLiquidityIndex
           userState.balance * currentLiquidityIndex
           "scaledBalance"   * "latest liquidity Index"
```

* user deposits 1000 DAI, when Index = 1.1
* scaledBalance = 909
* \_userState\[account].balance = 909

### AToken: Incrementation

The `scaledBalance` of a user is fixed, while the `currentLiquidityIndex` increments with every state-changing transaction, reflecting accrued interest. Due to this characteristic of the index, users' token balance increases with no action on their part.

{% hint style="info" %}
You can see your AToken balance increase in real-time directly in your wallet.
{% endhint %}

### AToken: mint

Earlier, we stated that instead of the deposit amount, the scaled amount is passed into the mint function. We will understand why from this section.&#x20;

**Execution flow**

AToken::`mint` -> ScaledBalanceTokenBase::`_mintScaled` ->  MintableIncentivizedERC20::`_mint`

<figure><img src=".gitbook/assets/image (177).png" alt=""><figcaption><p>AToken inherits ScaledBalanceTokenBase</p></figcaption></figure>

The abovementioned scaling can be seen from the first line in `_mintScaled`. Subsequently, `amountScaled` is passed into `_mint`, which increments the user's balance which is captured in `_userState[account].balance`.

<figure><img src=".gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

### Execution flow: Visual Aid

Here is a more complete execution flow chart spanning the relevant portions across different contracts.

<img src=".gitbook/assets/file.excalidraw (16).svg" alt="" class="gitbook-drawing">
