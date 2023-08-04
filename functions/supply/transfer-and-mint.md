# transfer & mint

## Overview

<figure><img src="../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [x] validateSupply
* [x] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer & mint
* [ ] If isFirstSupply

{% hint style="info" %}
updateInterestRates was covered in the common functions section&#x20;
{% endhint %}

### transfer

{% code overflow="wrap" %}
```solidity
IERC20(params.asset).safeTransferFrom(msg.sender, reserveCache.aTokenAddress, params.amount);
```
{% endcode %}

This transfers the asset to be supplied from user to the corresponding AToken contract

* from: `msg.sender`
* to: `aTokenAddress`  (aDAI, aWETH...)

{% hint style="info" %}
User' supplied assets are held on their respective AToken contracts; DAI held on aDAI.&#x20;
{% endhint %}

{% hint style="info" %}
**Use of safeTransferFrom**

`SafeERC20` is a helper to make safe the _interaction_ with _someone else’s_ ERC20 token, in your contracts. What the helper does for you is:

1. check the boolean return values of ERC20 operations and revert the transaction if they fail,
2. at the same time allowing you to support some non-standard ERC20 tokens that don’t have boolean return values.

It additionally provides helpers to increase or decrease an allowance, to mitigate [an attack 118](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729) possible with vanilla `approve`.\


For more: [https://forum.openzeppelin.com/t/making-sure-i-understand-how-safeerc20-works/2940](https://forum.openzeppelin.com/t/making-sure-i-understand-how-safeerc20-works/2940)
{% endhint %}

### mint

* minting of ATokens corresponding the supplied asset

{% code overflow="wrap" %}
```solidity
bool isFirstSupply = IAToken(reserveCache.aTokenAddress).mint(msg.sender, params.onBehalfOf, params.amount, reserveCache.nextLiquidityIndex);
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

* `mint` calls `_mintScaled`
* `_mintScaled` mints tokens in-line with the current liquidity index, as seen by `amountScaled`
* `amountToMint`: `amountScaled`cast as `uint128`
* `balanceIncrease`: interest accrued from user's last recorded index and current index&#x20;
* `_userState[onBehalfOf].additionalData` is updated with the latest index.

`balanceIncrease` is calculated as part of the mint event emission.

<figure><img src="../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

#### On `super.BalanceOf` and `_userState[onBehalfOf].additionalData`

* `super.BalanceOf` calls `balanceOf` declared in IncentivizedERC20.sol
* Inheritance: ScaledBalanceTokenBase > MintableIncentivizedERC20 > IncentivizedERC20

<figure><img src="../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

* notice the irregular definition of `balanceOf`, it calls out a mapping `_userState`
* `_userState` is mapping of user addresses to `UserState` structs
* Each `UserState` struct stores a user's balance with a wildcard field of `additionalData`

{% hint style="info" %}
`additionalData` in the context of Atokens

* used to store the index of the user's last "index-sensitive" action: --> last supply/withdrawal/borrow/repayment
{% endhint %}

#### Returns

`mint` returns the bool value of `(scaledBalance == 0)`

* `scaledBalance` is the amount of balance a user has of a specific asset pre-dating this supply action
* `(scaledBalance == 0)` will return true only if this supply action is the user's very first supply action - specific to said asset.
* The return bool value is useful to us as explained in the next section.&#x20;

### Visual Aid

<img src="../../.gitbook/assets/file.excalidraw (15).svg" alt="" class="gitbook-drawing">
