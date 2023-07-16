# validateSupply

## Overview

<figure><img src="../../.gitbook/assets/image (73) (1).png" alt=""><figcaption></figcaption></figure>

* [x] <mark style="color:orange;">cache</mark>
* [x] <mark style="color:orange;">updateState</mark>
* [ ] validateSupply
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer & mint
* [ ] If isFirstSupply

{% hint style="info" %}
`cache`,`updateState`, `updateInterestRates` were explained in the common functions section.
{% endhint %}

## validateSupply

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

<img src="../../.gitbook/assets/file.excalidraw (16).svg" alt="" class="gitbook-drawing">

In `validateSupply`, two key checks are being executed:

1. **check asset status**: ensure ACTIVE & ensure NOT FROZEN & NOT PAUSED
2. **check supply cap:** ensure it is not exceeded by this new supply action

### getFlags

Function takes `ReserveConfigurationMap` struct as parameter. `ReserveConfigurationMap` contains `data`, which is a bitmap, and the various status flags range from bit 56 to 60.

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

* For in-depth explanation: see [getFlags](validatesupply.md#getflags)&#x20;
* Execution reverts if any of the following conditions are TRUE:
  * asset is not ACTIVE
  * asset is PAUSED
  * asset is FROZED

### Check supply cap

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

The long require statement simply means:

```solidity
require(supplyCap == 0 || newSupply <= supplyCap)
```

* if supplyCap == 0, require statement clears _(unlimited supply allowed)_
* if newSupply <= supplyCap, require statement clears

The easier (less-gas intensive) check is placed first, to allow early reversion thereby saving gas on wasted calculations.

#### getSupplyCap

<figure><img src="../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

```solidity
uint256 internal constant SUPPLY_CAP_START_BIT_POSITION = 116
uint256 internal constant SUPPLY_CAP_MASK = 0xFFFFFFFFFFFFFFFFFFFFFFFFFF000000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
```

* Only bits 116 - 151 are set to `0` in the mask; these bits will be set to `1` after the `~` operation.
* `data & ~SUPPLY_CAP_MASK` will return a hexadecimal number in which all the characters are set to `0,`except potentially the bits representing the supply cap.
* These bits will return the hex value of the supply cap.
* If supply cap is indeed `0`, then `0x000...000` would be returned.

Numbers in Solidity are left-padded. Therefore to convert our resulting hexadecimal number to the appropriate number representation so that it can be cast as a `uint256`, right shifting by 116 bits is required.

* this will remove all the 0s on the right, from the least significant bit to the value
* left-padding will be done accordingly, to ensure 32 byte representation

{% hint style="info" %}
See a more in-depth walkthrough of this process at [getReserveFactor](../../primer/bitmap-and-masks/#getreservefactor).
{% endhint %}

#### newSupply

```solidity
// newSupply: currentSupply + amount
AToken.scaledTotalSupply + [accruedToTreasury * nextLiquidityIndex] + amount
```

* `amount` is the incoming injection of deposits
* `AToken.scaledTotalSupply` represents supply held **only by suppliers**
* Need to account for treasury portion separately as Treasury's ATokens are not minted, just numerically tracked as we saw in `.updateState`&#x20;
