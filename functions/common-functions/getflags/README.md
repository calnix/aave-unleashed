# getFlags

## getFlags

This function is defined in ReserveConfiguration.sol.

Function takes `ReserveConfigurationMap` struct as parameter. As we can see below, `ReserveConfigurationMap` contains `data`, which is a bitmap, and the various status flags range from bit 56 to 60.

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption><p>getFlags</p></figcaption></figure>

`getFlags` makes use of the following bitmap masks to extract the relevant bits pertaining to each status flag:

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption><p>masks</p></figcaption></figure>

The masks are applied upon the bitmap contained within the ReserveConfigurationMap struct. The bits that are relevant to us are from bit 56 - 60.

{% hint style="info" %}
Notice how only the 15th hex character from the least significant bit varies in value across each of the masks - all other characters are `F`.&#x20;

The exception to this is the PAUSED\_MASK.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption><p>DataTypes.sol</p></figcaption></figure>

* All the bits that are irrelevant within the bitmap have their corresponding bit in the bitmap mask to be set to `F` (`1111` in binary).
* The bitwise NOT (`~`), will led to an inversion of bit values, and together with the bitwise AND (`&`), this means that irrelevant bits will be paired up against `0` values in the mask for the `&` operation.&#x20;
* As such, they will be set to `0`. The only bits that have a chance to be **non-zero**, will fall within bit 56 - 60, surfacing the values we require.

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

### Example: ACTIVE\_MASK illustration

As an example, lets overlay the ACTIVE\_MASK upon the bitmap.

```solidity
// ACTIVE_MASK: 0xFFF...E...FF
0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFFFFFFFFFF
```

<figure><img src="../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

* First 14 hex characters from the least significant bit are set to `F` - covers the values of bits from 0 - 55.
* The 15th hex character in `ACTIVE_MASK` is set to `E`; `1110` in binary.
* This results in only bit 56 being set to `0` - consequently, through the paired bitwise NOT and AND operation, only this bit has a possibility of being any value other than `0`.

**If the active flag is set to 0, the result will be as follows:**

```cpp
            0x0000...0...0   (data)
         &  0x0000...1...0   (~mask)
          -------------
           0x00000...0...0   (result)

// Remember the rules of bitwise AND (&):
// 0 & 0 = 0
// 0 & 1 = 0
// 1 & 1 = 1
```

Finally, the bitwise result is checked if its NOT equal to 0:

```solidity
function getFlags(..) returns(...) {
    ...
    (dataLocal & ~ACTIVE_MASK) != 0,
    ...
}

// earlier result 
(0x00000...0...0) != 0
```

* result is `0x00....00`, which is equal to `0`, evaluating to be `FALSE`. Therefore, in this case, `isActive == FALSE`.

**On the other hand, if the active flag was set to `1` like so:**

```cpp
            0x0000...1...0   (data)
         &  0x0000...1...0   (~mask)
          -------------
           0x00000...1...0   (true)
```

* The bitwise result contains a single bit set to `1`, therefore it will not evaluate to be `0`.

```solidity
(dataLocal & ~ACTIVE_MASK) != 0,
(0x00000...1...0) != 0
```

* `isActive` in this scenario will be `true`.

### **Summary**

If the active flag bit achieves a **non-zero** value after the bitwise operations, `!= 0`, will evaluate to **true**, thereby indicating that asset is in **ACTIVE** status.

If the active flag bit has a **zero** value, `!= 0`, will evaluate to **false**,  indicating that the asset is **NOT ACTIVE.**

{% hint style="info" %}
This process applies similarly to the other status flags. If you are confused as to how bitwise operations work, please see the previous section: [bitmap & masks](../../../primer/bitmap-and-masks/).
{% endhint %}

## Overview of required checks

* Reference: [https://docs.aave.com/faq/frozen-markets-and-reserves](https://docs.aave.com/faq/frozen-markets-and-reserves)

<table data-full-width="true"><thead><tr><th width="269">Function</th><th width="107">Active</th><th width="95">Frozen</th><th width="105">Paused</th><th width="188">Borrowing Enabled</th><th>Stable Borrowing Enabled</th></tr></thead><tbody><tr><td>Supply</td><td>Yes</td><td>No</td><td>No</td><td></td><td></td></tr><tr><td>Withdraw</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>Borrow</td><td>Yes</td><td>No</td><td>No</td><td>Yes</td><td>Depends on the mode</td></tr><tr><td>Repay</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>SwapRateMode</td><td>Yes</td><td>No</td><td>No</td><td></td><td>Depends on the mode</td></tr><tr><td>RebalanceStableBorrowRate</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>SetUseReserveAsCollateral</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>Flashloan</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>Liquidate</td><td>Yes</td><td></td><td>No</td><td></td><td></td></tr><tr><td>AToken Transfer</td><td></td><td></td><td>No</td><td></td><td></td></tr></tbody></table>

{% hint style="info" %}
* In `V2` it is not possible to pause a specific reserve, only the entire pool.
* In `V3` role `emergencyAdmin` can pause a specific reserve.
{% endhint %}

