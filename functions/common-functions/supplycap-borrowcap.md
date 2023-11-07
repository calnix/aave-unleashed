# SupplyCap, BorrowCap

## TLDR

**Borrow caps:** limits of how much of each asset can be borrowed, which reduces insolvency risk.

**Supply caps**: limits how much of a certain asset is supplied. Helps control excessive exposure to a certain asset and mitigate attacks like infinite minting or price oracle manipulation.

{% hint style="info" %}
If borrow/supply cap is set to 0; this sigifies that there is no cap.
{% endhint %}

{% hint style="success" %}
In case Borrow Cap of the reserve is set lower than the current totalDebt, the existing borrowers are not effected but no more borrows (stable or variable) can be initiated for that reserve.
{% endhint %}

{% hint style="success" %}
In case _Supply Cap_ of the reserve is set lower than the current liquidity of the reserve, the existing suppliers are not effected but no more liquidity can be supplied to that reserve.
{% endhint %}

### How are the caps stored

Each asset has a corresponding `ReserveData` struct which outlines its key information and configuration.&#x20;

* `PoolStorage` holds mapping `_reserves` which maps asset address to `ReserveData`.

Within each`ReserveData` struct, there is a nested struct of type `ReserveConfigurationMap`. The nested struct contain a bitmap, storing the asset's configuration:

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption><p><code>ReserveConfigurationMap</code></p></figcaption></figure>

An asset's borrow and supply caps are defined within the bitmap, along with other relevant information.&#x20;

### getSupplyCap

To retrieve a specific asset's supply cap, we call `getSupplyCap` and pass said asset's `ReserveConfigurationMap`.

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

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
See more at [getReserveFactor](../../primer/bitmap-and-masks/#getreservefactor)
{% endhint %}

### getBorrowCap

```solidity
uint256 internal constant BORROW_CAP_START_BIT_POSITION = 80
uint256 internal constant BORROW_CAP_MASK = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF000000000FFFFFFFFFFFFFFFFFFFF;
```

* Process is similar to getSupplyCap
