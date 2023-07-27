# WadRayLibrary

## Fixed-point Math

There are only two numerical data types in Ethereum:

* signed integers, int256
* unsigned integers, uint256

There is no support for fractional arithmetic. Hence, to support decimals places and such, is the use of fixed-point numbers. These are basically simple fractions whose denominator is a predefined constant, usually a power of 2 (“binary”) or 10 (“decimal”). The standard choices of denominator in the decimal case are 10¹⁸ (“wad”) or 10²⁷ (“ray”).

### Wad and Ray

Presently, solidity compiler doesn’t support fixed-point mathematics yet. Hence, it is not possible to represent the number `3.1415` natively as a fixed-point number. The common workaround is by representing decimal numbers as unsigned integers and specifying the level of precision that is being used alongside it.&#x20;

```solidity
  uint256 internal constant WAD = 1e18;
  uint256 internal constant RAY = 1e27;
```

* `wad` is a decimal number with **18** digits of precision that is being represented as an integer.
* `ray` is a decimal number with **27** digits of precision that is being represented as an integer.

```solidity
// representing 121.234 as a wad
uint tokenBalance = 121234000000000000000  
uint decimals = 18

// representing 4 as a wad
uint fourAsWad = 4 * 1e18 
```

Using wad and rays are pretty straightforward for addition and subtraction, but when performing multiplication or division one must rescale the result to get the right answer.

**Example**

```solidity
1.5 * 2.7 == 4.05

//Regular integer arithmetic adds orders of magnitude:
150 * 270 == 40500

// Wad arithmetic does not add orders of magnitude:
wadMul(1.5 ether, 2.7 ether) == 4.05 ether
```

#### The scaling required when multiplying and dividing wads is as follows:

```solidity
// multiplication
wadMul = wadA * wadB / WAD

// division
wadDiv = wadA * WAD / wadB
```

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
Division should always be done last to preserve precision
{% endhint %}

Note that in solidity natively rounds down; therefore in our examples of mul and div, our results were rounded down. We will see how to WadRayLibrary employs mul and div functions that allow for **rounding to the nearest integer**.

## **WadRayLibrary: Mul & div w/ rounding**

### Rounding in solidity

All integer division **rounds down** to the nearest integer.

```solidity
// Result is 2 [2.5 does not round up to 3]
uint result = 5 / 2; 

// Result is 3 [3.75 does not round up to 4]
uint result = 15 / 4;
```

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

So we cannot natively rely on Solidity for rounding to the nearest integer, or similar use cases. Hence, the use of WadRayLibrary in Aave.&#x20;

* Provides mul and div function for wads (decimal numbers with 18 digits of precision) and rays (decimal numbers with 27 digits of precision)
* Operations are rounded. If a value is **>=** 0.5, will be rounded up, otherwise rounded down

### Rounding in WadRayMath&#x20;

* Rounding deals with fractional values.&#x20;
* Since solidity always rounds down, we can allow for rounding to the nearest integer by adding half

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* if remainder < 0.5 -> remainder +0.5 < 1  => **rounded down to 0**
* if remainder $$\geq$$ 0.5 ->  remainder + 0.5 $$\geq$$ 1  => **rounded up to 1**

**However, we cannot simply apply this as:**&#x20;

```solidity
wadDiv = a / b + 0.5
```

As the rounding would have already occurred as part of the division operation. We need to repackage this such that the division occurs last:

<figure><img src="../../.gitbook/assets/image (83).png" alt="" width="530"><figcaption><p>general form: division</p></figcaption></figure>

{% hint style="success" %}
**General Form**

* To allow for rounding to nearest integer, add **half of the divisor** to the dividend.&#x20;
{% endhint %}

Let's connect this understanding with the implementation of `wadDiv` in the library.

### wadDiv

* divides two wads, _rounding half_ _wad and above_ _up to the nearest wad_.

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption><p>wadDiv</p></figcaption></figure>

**The division occurs at:**

```solidity
c := div(add(mul(a, WAD), div(b, 2)), b)

// translating to a more readable form:
c = a / b = [(a * WAD) + (b/2)] / b
```

* Multiply a WAD to negate the WAD removal from division. This ensures that WAD representation is adhered to.&#x20;
* Add half of the divisor (`b / 2`) to the dividend, `a` to allow for rounding to the nearest integer .
* The addition of half b was explained earlier.&#x20;

**Overflow check**

* To avoid overflow, `a <= (type(uint256).max - HALF_WAD) / b`
* The check is executed first, so that if it fails, transaction can revert early saving on gas.
* The `if or()` condition is required to account for the possibility that the divisor, b, is 0.

{% hint style="info" %}
* div by zero in Yul returns `0` -> `a/0 = 0`. This should revert, but does not.&#x20;
{% endhint %}

```solidity
// if b is 0 revert OR overflow condition true, revert:
if or(iszero(b), iszero(iszero(gt(a, div(sub(not(0), div(b, 2)), WAD))))) {
        revert(0, 0)
}

// overflow condition: a <= (type(uint256).max - HALF_WAD) / b
iszero(iszero(gt(a, div(sub(not(0), div(b, 2)), WAD))))
```

This condition prevents the result of the multiplication from exceeding the maximum value that can be represented by a `uint256`.

* Both `a` and `b` are passed into the function as uint256 parameters; so we do not check them.
* However, some **manipulations are applied to the dividend**, as part of our rounding effort and WAD representation:
  * `dividend, A`: `[(a * WAD ) + (b/2)]`, for A/b
  * &#x20;we need to ensure that A does not overflow&#x20;
  * when using assembly, we must apply {over,under}flow checks ourselves.

{% hint style="success" %}
[Since Solidity v0.8](https://docs.soliditylang.org/en/v0.8.3/080-breaking-changes.html), the compiler has checks for {over,under}flow by default for all arithmetic operations. These checks do not apply to assembly ([yul](https://docs.soliditylang.org/en/v0.8.3/yul.html)) arithmetic operations.
{% endhint %}

{% hint style="info" %}
The bitwise negation operation, denoted as `not`, flips all the bits in the binary representation of the given value.

* `not(0)` flips all the bits of `0`, resulting in a value where all bits are set to `1`
* `not(0)` in YUL would be equivalent to `2^256 - 1`&#x20;
* **`not(0) => type(uint256).max`**
{% endhint %}

### wadMul

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

**The multiplication occurs at:**

```solidity
c := div(add(mul(a, b), HALF_WAD), WAD)

// translating to a more readable form:
c = [(a * b) + WAD/2] / WAD
```

* `WAD / 2`: Add half of the divisor to the dividend, to allow for rounding to the nearest integer.
* Divide by WAD to negate the additional WAD from multiplication. This ensures that WAD representation is adhered to.&#x20;

{% hint style="warning" %}
The above applies to rays just the same, as such we will not go through them in detail.
{% endhint %}

## **Converting between Wad and Ray**

* wad: 1e18, ray: 1e27
* difference: 1e(27-9) = **1e9**

### **wad to ray**

Scaling a wad up to ray is easy: simply multiply by the difference in precision:

```solidity
uint256 internal constant WAD_RAY_RATIO = 1e9;
```

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption><p>WadRayMath.sol</p></figcaption></figure>

**All the overflow check does is ensure that** `b/WAD_RAY_RATIO == a`, returns 1, which would be representative of true.

* If `b/WAD_RAY_RATIO`**`!=`**`a`,  the function will revert as there has been overflow

### **ray to wad**

Scaling a ray down involves some loss of precision, as we move from 27 dp to 18 dp.

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

1. reduce the given ray by `1e9` to obtain a wad: 1e27 -> 1e18
2. get the remainder of `ray/1e9` via modulo operation&#x20;
3. rounding to nearest wad:&#x20;
   1. if `remainder` $$\geq$$ `1e9/2`, add 1 to result **`b`**

{% hint style="info" %}
Use of `iszero` + `lt` to represent $$\geq$$
{% endhint %}

**Remember, when dividing and wanting to achieve rounding to the nearest integer (wad/ray), we need to add half of the divisor to the dividend, as explained earlier.**&#x20;



{% hint style="success" %}
**From** [**Aave-v2-whitepaper**](https://github.com/aave/protocol-v2/blob/master/aave-v2-whitepaper.pdf)**:**

* The V1 WadRayMathlibrary internally uses SafeMath to guarantee the integrity of operations.&#x20;
* After an in depth analysis, SafeMath incurred intensive high costs in critical areas of the protocol, with a 30 gas fee for each call.&#x20;
* This supported the refactoring of WadRayMath to remove SafeMath, which saves 10-15k gas on some operations.
{% endhint %}

