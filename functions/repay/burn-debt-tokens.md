# burn debt tokens

## Overview

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache + updateState</mark>
* [x] get current debt&#x20;
* [x] validateRepay
* [x] get paybackAmount
* [ ] **burn debt tokens**
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] setBorrowing
* [ ] updateIsolatedDebtIfIsolated
* [ ] handle repayment

Previously, we established how much debt user was inclined repay and assigned its value to **`paybackAmount`**. Now we are going to burn the corresponding debt tokens to reflect a repayment in debt.

## Burn

Depending on which kind of debt the user has, stable or variable, the burn function will be called on the corresponding debt token contract.

<img src="../../.gitbook/assets/file.excalidraw (30).svg" alt="" class="gitbook-drawing">

## Burn stableDebtToken

<figure><img src="../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

* See StableDebtToken: [burn](../../contracts/stabledebttoken/#burn)

## Burn variableDebtToken

* `burn` calls `_burnScaled` on  VariableDebtToken.sol.
* `_burnScaled` scales the burn amount against `variableBorrowIndex` and calls `_burn` passing the `amountScaled`.
* `_burn` is defined on MintableIncentivizedERC20.sol:
  * reduces total supply
  * reduces user's balance

<img src="../../.gitbook/assets/file.excalidraw (9).svg" alt="" class="gitbook-drawing">

**`_burnScaled`**&#x20;

* scale burn amount against `variableBorrowIndex`
* `balanceIncrease`: update user's variable debt balance to bring it in-line with latest index
* `additionalData` contains the index at which the balance was last updated.
  * this is referenced to calculated `balanceIncrease`, then subsequently updated
* call `_burn`, passing scaled amount
* if `balanceIncrease` > `amount` :&#x20;
  * interest accrued outweighs the burn amount
  * emit mint and transfer
* Else `amount` > `balanceIncrease` :
  * burn amount outweighs interest accrued
  * emit burn and transfer

{% hint style="info" %}
**Note**: \
In`balanceIncrease` > `amount`, tokens are not actually minted. `amountToMint` is simply a representation of the remaining interest accrued, less the nominal burn amount.&#x20;
{% endhint %}

Essentially, each unit of scaled token represents itself and the interest it has accrued as dictated by the index. Therefore, to account for the interest, the nominal amount is divided by the index to obtain **`amountScaled`**.

* Index here refers to **variableBorrowIndex**

<details>

<summary>Example</summary>

Assume 100 DAI to be burnt:

* amount = 100 DAI
* variableBorrowIndex = 1.1
* amountScaled = 100 / 1.1 = 90.90

With index at 1.1, each token has an interest premium of 10%. So 100 debtTokens is actually worth 110 DAI.

Therefore an estimate of 90.90 debtTokens account for 100 DAI worth of debt.&#x20;

</details>

{% hint style="info" %}
**Note**: the actual number of tokens being burnt is **`amountScaled`**. This is because debt tokens accrue interest in the same way that ATokens do; against an index.
{% endhint %}

**`_burn`**

* reduce total supply by burn amount
* get user's balance decrement it by burn amount
* apply incentives if defined (`.handleAction`)
