# Bitmap & Masks

## What is a bitmap

A bitmap refers to a data structure that uses a sequence of bits to represent the status or presence of certain elements or flags. It is commonly used to efficiently store and manipulate binary data, such as flags, permissions, or states.

In Solidity, a bitmap is typically represented using an unsigned integer (uint256), where each bit within the integer represents a specific element or flag. Each bit can be either set (1) or unset (0), indicating the presence or absence of a particular attribute or state.

<figure><img src="../../.gitbook/assets/image (59).png" alt="" width="375"><figcaption></figcaption></figure>

By using bitwise operations, such as bitwise AND (&), bitwise OR (|), bitwise XOR (^), and bitwise negation (\~), it is possible to manipulate individual bits within the bitmap efficiently. These operations allow for checking and setting specific bits, querying the status of flags, and performing other bitwise operations.

{% hint style="info" %}
For a uint256 bitmap, imagine that there are 256 light bulbs in total. Each light bulb representing 1 bit. Each switch can either be off (set to `0`) or on (set to `1`).&#x20;

* each bulb represents a state
* e.g. is an asset active
* if the bulb is set to `1` -> active
{% endhint %}

## Why use bitmaps&#x20;

Using bitmaps can provide several benefits in terms of storage efficiency and gas efficiency.

1. **Compact Storage**: Bitmaps allow for efficient storage of multiple boolean or binary attributes within a single variable. Instead of using separate boolean variables or larger data types, a bitmap can represent multiple flags or states using a minimal amount of storage.
2. **Gas Efficiency:** Working with bitmaps can lead to gas savings in contract execution. Since bitwise operations operate on the binary representation of the data, they are generally less costly in terms of gas consumption compared to other operations like conditional statements or looping through individual flags.

### Aave's bitmaps

Aave uses two types of bitmaps:

* ReserveConfiguration: to track information related to each asset within Aave
* UserConfiguration: to track which assets a user is supplying as collateral and borrowing

{% hint style="info" %}
**From** [**Aave-v2-whitepaper**](https://github.com/aave/protocol-v2/blob/master/aave-v2-whitepaper.pdf)**:**

In the initial V1 release, the protocol loops through all the active assets to identify the user deposits and loans. This resulted in high gas consumption and reduced scalability - as the cost of withdrawing/borrowing/repaying/liquidating assets would increase as more assets are listed on the protocol.
{% endhint %}

**UserConfiguration**

<figure><img src="../../.gitbook/assets/image (19).png" alt="" width="554"><figcaption><p>Figure 4: UserConfiguration Bitmask</p></figcaption></figure>

The bitmask has a 256 bit size, it is divided in pairs of bits, one for each asset. The first bit of the pair indicates if an asset is used as collateral by the user, the second whether an asset is borrowed by the user.&#x20;

**Advantages**

* If you need to traverse the list of assets for a user, to check what is being borrowed, just need to sload the bitmap to memory once; then read from memory.
* Able to do complex queries cheaply and efficiently.
* Especially useful for checking isolation and e-mode -> if certain assets are being used by the user.

&#x20;**Disadvantages**:&#x20;

* Only 128 assets can be supported, to add more, another uint256 needs to be used.&#x20;
* For the calculation of the account data, the protocol would still need to query all listed assets.

{% hint style="info" %}
An 128 asset limit is acceptable. Considering the case where a user is supplying a huge range of assets; it would be gas intensive to traverse the entire list to calculate his position health.
{% endhint %}

#### ReserveConfiguration

<figure><img src="../../.gitbook/assets/image (1) (3).png" alt=""><figcaption><p>ReserveConfigrationMap</p></figcaption></figure>

A bitmask has also been introduced to store the reserve configuration, defined in figure 5. A similar packing could have been achieved by using uint32 and booleans, the bitmask benefits from more gas efficiency, and more so when updating multiple configurations at once.

{% hint style="info" %}
If there is a need to store more data, this bitmask can be extended by adding another uint256 into the struct.

* The bitmap is held within a struct that contains a single uint256 variable.
{% endhint %}

## Bitwise Operators

### `&`: AND

<figure><img src="../../.gitbook/assets/image (166).png" alt="" width="563"><figcaption></figcaption></figure>

The bitwise AND operator ( `&` ) compares each bit of the first operand to the corresponding bit of the second operand.&#x20;

* If both bits are 1, the corresponding result bit is set to 1.
* Else, the result bit is set to 0.&#x20;

```solidity
// x     = 1110 = 8 + 4 + 2 + 0 = 14
// y     = 1011 = 8 + 0 + 2 + 1 = 11
// x & y = 1010 = 8 + 0 + 2 + 0 = 10
function and(uint x, uint y) external pure returns (uint) {
    return x & y;
}
```

### `~`: NOT

```solidity
// x  = 00001100 =   0 +  0 +  0 +  0 + 8 + 4 + 0 + 0 = 12
// ~x = 11110011 = 128 + 64 + 32 + 16 + 0 + 0 + 2 + 1 = 243
function not(uint8 x) external pure returns (uint8) {
    return ~x;
}
```

The bitwise NOT operator ( `~` ) inverts the bits of the binary sequence.&#x20;

* input bit 1 -> 0
* input bit 0 -> 1

{% hint style="info" %}
This is also known as One's compliment&#x20;
{% endhint %}

## `<<` and `>>`: Left shift and right shift <a href="#left-shift-and-right-shift-operators-and" id="left-shift-and-right-shift-operators-and"></a>

The bitwise shift operators move the bit values of a binary object, by the defined number of bits.

#### Left Shifts (`<<`)

When shifting left, the most-significant bit is lost, and a `0` bit is inserted on the other end. Essentially, all the bits take a step to the left, with the MSB falling off; right-padded with `0`.

```solidity
0010 << 1  →  0100    //left-shift by 1 bit
0010 << 2  →  1000    //left-shift by 2 bits
```

A single left shift **multiplies** a binary number by 2:

```solidity
0010 << 1  →  0100

0010 is 2
0100 is 4
```

#### Right Shifts (>>)

When shifting right, the least-significant bit is lost and a `0` is inserted on the other end. Essentially, all the bits take a step to the right, with the LSB falling off; left-padded with `0`.

```solidity
1011 >> 1  →  0101
1011 >> 3  →  0001
```

For positive numbers, a single logical right shift **divides** a number by 2, throwing out any remainders.

```solidity
0101 >> 1  →  0010

0101 is 5
0010 is 2
```

#### Solidity example

```solidity
// 1 << 0 = 0001 --> 0001 = 1
// 1 << 1 = 0001 --> 0010 = 2
// 1 << 2 = 0001 --> 0100 = 4
// 1 << 3 = 0001 --> 1000 = 8
// 3 << 2 = 0011 --> 1100 = 12
function shiftLeft(uint x, uint bits) external pure returns (uint) {
    return x << bits;
}

// 8  >> 0 = 1000 --> 1000 = 8
// 8  >> 1 = 1000 --> 0100 = 4
// 8  >> 2 = 1000 --> 0010 = 2
// 8  >> 3 = 1000 --> 0001 = 1
// 8  >> 4 = 1000 --> 0000 = 0
// 12 >> 1 = 1100 --> 0110 = 6
function shiftRight(uint x, uint bits) external pure returns (uint) {
    return x >> bits;
}
```

## Applications

Aave utilizes 2 bitmaps, **ReserveConfigurationMap** and **UserConfigurationMap**, both defined within the library `DataTypes`.

### **ReserveConfigurationMap**&#x20;

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption><p>DataTypes.sol</p></figcaption></figure>

Let's examine how reserve factor (bit 64-79) can be retrieved from this bitmap.

### getReserveFactor

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption><p>ReserveConfiguration.sol</p></figcaption></figure>

{% code fullWidth="true" %}
```solidity
uint256 internal constant RESERVE_FACTOR_MASK = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0000FFFFFFFFFFFFFFFF;
uint256 internal constant RESERVE_FACTOR_START_BIT_POSITION = 64;
```
{% endcode %}

The mask is a 64 digit hexadecimal number, representing 256 bits.&#x20;

* each hexadecimal digit represents 4 bits => 64 \* 4 = 256 bits
* first 64 digits are set to `1`, followed by 16 bits set to `0`, then 176 bits set to `1`
* remember, binary is read from right to left. The rightmost place number being zero.

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

</div>

And that is how the reserve factor is extracted from the bitmap as a uint256 value.

### **UserConfigurationMap**

* consider **isUsingAsCollateralOrBorrowing**
