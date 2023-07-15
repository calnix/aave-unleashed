# Cleanup + Collect repay

## Overview

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache + updateState</mark>
* [x] get current debt&#x20;
* [x] validateRepay
* [x] get paybackAmount
* [x] burn debt tokens
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [ ] **setBorrowing**
* [ ] **updateIsolatedDebtIfIsolated**
* [ ] **handle repayment**

We will cover the last three components in this section.

<figure><img src="../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

### setBorrowing

If there is no outstanding debt left, within userConfig, set the borrowing flag for the asset to `0`.

* For function's inner workings, see: [setBorrowing](../borrow/setborrowing.md#setborrowing) under the borrow section.

### **updateIsolatedDebtIfIsolated**

In the event that the debt position was collateralized by an isolated asset, and a repay or liquidation occurred

* total debt of the isolated asset must be decremented (`isolationModeTotalDebt`)
* [getisolationModeState](../borrow/getisolationmodestate.md#getisolationmodestate) was previously explained in the borrow section

<img src="../../.gitbook/assets/file.excalidraw (18).svg" alt="" class="gitbook-drawing">

**If user is in isolation mode:**

* get total isolated debt from `reservesData` (`isolationModeTotalDebt`)
* rebase the decimals of `paybackAmount`

**`if isolationModeTotalDebt < repayAmount`**

* reset total isolated debt on asset to `0`
* Else: decrement total isolated debt by `repayAmount`

## Collect repay

Finally, the repayment is collected from the function caller.&#x20;

<figure><img src="../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

**If `useATokens` was set to `true`, like in `repayWithATokens`, Atokens would be burnt.**&#x20;

Example_:_ User has DAI debt and also holds aDAI token

* can use aDAI to repay DAI debt in single transaction without any approvals or having to withdraw their supplied liquidity to the pool using `repayWithATokens` feature
* aDAI tokens would be burnt, reducing his claim on supplied assets.
* See withdraw section for [AToken.burn](../withdraw/burn-atokens.md)

**Else, `useATokens` is set to `false`, in the case of `repay`;**&#x20;

* asset is pulled from user via `safeTransferFrom`
* example: borrowed USDC, repaying USDC: principal + plus interest
