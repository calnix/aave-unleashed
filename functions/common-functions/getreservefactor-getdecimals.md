# getReserveFactor, getDecimals

## getReserveFactor

<figure><img src="../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

* `getReserveFactor`, returns the value of `reserveFactor` as a uint256 variable.
* `reserveFactor`, amongst other data, is stored within the bitmap  `ReserveConfigurationMap`

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
The reserve factor is a percentage of protocol interest which goes to the Aave Ecosystem Reserve. The rest of the borrow interest goes to the suppliers.
{% endhint %}

#### It does 3 things:

* Bitwise NOT of RESERVE\_FACTOR\_MASK: **`~`**`RESERVE_FACTOR_MASK`
* Bitwise AND: `data`` `**`&`** **`~`**`RESERVE_FACTOR_MASK`
* Right shift the result of the above by `RESERVE_FACTOR_START_BIT_POSITION`

#### RESERVE\_FACTOR\_MASK

```solidity
// 64 characters (we ignore the leading 0x)
0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0000FFFFFFFFFFFFFFFF
```

This is a 256-bit hexadecimal number, represented by 64 hexadecimal characters.&#x20;

* each hex character represents 4 bits => 4 \* 64 = 256 bits

In binary: first 176 bits are all `1` (represented by `F` in hexadecimal), followed by 16 zeros, and ending with another 64 bits of `1`.

{% hint style="info" %}
`0xF` can be directly interpreted in binary as such:&#x20;

`F` = 15 = 8 + 4 + 2 + 1 = `1111`
{% endhint %}

#### **`~`**`RESERVE_FACTOR_MASK`

The tilde is the bitwise NOT operator -> it flips the bits, changing 0s to 1s and 1s to 0s.

<figure><img src="../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

* Notice that only the bits 64-79 are now `F`, which corresponds to the range of bits that contain the `reserveFactor` value as seen in `ReserveConfigurationMap`
* #### **`~`**`RESERVE_FACTOR_MASK: 0x00..FFFF..00` &#x20;

#### `data`` `**`&`** **`~`**`RESERVE_FACTOR_MASK`

* `&` is the operator for bitwise AND operation
* AND operation returns `1` only if both the bits are `1`.
* `data AND 0x00..FFFF..00:`&#x20;
  * this means that it is only possible for the result of the operation to contain 1s within the range of 64-79
  * all other bits will be zero, as a bitwise ADD operation with a 0 will result in 0.
  * so we have the possibility of 1s occurring within 64-79 bit, if the corresponding bit in self.data, is also 1.

{% hint style="info" %}
The purpose of the mask is to wipe out all other irrelevant data, while extracting only the relevant portion of the bitmap
{% endhint %}

#### Right shift ( >> )

To illustrate this portion, let us assume the result so far is as follows:

```
0x00000000000000000000000000000000000000000000 F0FF 0000000000000000 
```

* The result is right shifted by `RESERVE_FACTOR_START_BIT_POSITION`
* `RESERVE_FACTOR_START_BIT_POSITION = 64`

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

We see that the trailing zeros are dropped off, and replaced as leading zeros. Essentially, the section of bits corresponding to `reserveFactor` is moved all the way to the end.

**Why?**

**In short, right shifting such that the trailing zeros are removed so that the value is left-padded, results in the proper representation of a number.**

In Solidity, strings and bytes are padded on the lower-order (right) side with zero-bytes, while other types (such as numbers and addresses) are padded on the higher-order side.

As an example, this is how we would store the string "abcd" in one full word (32 bytes):

`0x6162636400000000000000000000000000000000000000000000000000000000`

This is how the number `0x61626364` would be stored:

`0x0000000000000000000000000000000000000000000000000000000061626364`

{% hint style="info" %}
uint256 (and other numbers) are left-padded.
{% endhint %}

## getDecimals

<figure><img src="../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

* Process is similar to **getReserveFactor**
