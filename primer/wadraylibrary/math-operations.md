# Math Operations

## Rounding in solidity

All integer division **rounds down** to the nearest integer.

```solidity
// Result is 2 [2.5 does not round up to 3]
uint result = 5 / 2; 

// Result is 3 [3.75 does not round up to 4]
uint result = 15 / 4;
```

<figure><img src="../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

So we cannot natively rely on Solidity for rounding to the nearest integer, or similar use cases. Hence, the use of WadRayLibrary in Aave.&#x20;

## WadRayMath library: mul and div

* Provides mul and div function for wads (decimal numbers with 18 digits of precision) and rays (decimal numbers with 27 digits of precision)
* Operations are rounded. If a value is **>=** 0.5, will be rounded up, otherwise rounded down

### Rounding in WadRayMath&#x20;

* Rounding deals with fractional values.&#x20;
* Since solidity always rounds down, we can allow for rounding to the nearest integer by adding half

<figure><img src="../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

* if remainder < 0.5 -> remainder +0.5 < 1  => **rounded down to 0**
* if remainder $$\geq$$ 0.5 ->  remainder + 0.5 $$\geq$$ 1  => **rounded up to 1**

**However, we cannot simply apply this as:**&#x20;

```solidity
wadDiv = a / b + 0.5
```

As the rounding would have already occurred as part of the division operation. We need to repackage this such that the division occurs last:

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
General Form: To allow for rounding to nearest integer, add half of the divisor to the dividend&#x20;
{% endhint %}

Let's connect this understanding with the implementation of wadDiv in the library.

### wadDiv

* divides two wads, _rounding half_ _wad and above_ _up to the nearest wad_.

<figure><img src="../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

**The division occurs at:**

```solidity
c := div(add(mul(a, WAD), div(b, 2)), b)

// translating to a more readable form:
c = a / b = [(a * WAD) + (b/2)] / b
```

* Multiply a WAD to negate the WAD removal from division. This ensures that WAD reprsentation is adhered to.&#x20;
* Add half of the divisor (`b / 2`) to the dividend, `a` to allow for rounding to the nearest integer .
* The addition of half b was explained earlier.&#x20;

**Overflow check**

* To avoid overflow, `a <= (type(uint256).max - HALF_WAD) / b`
* The check is executed first, so that if it fails, transaction can revert early saving on gas.
* The `if or()` condition is required to account for the possibility that the divisor, b, is 0.

{% hint style="info" %}
div by zero in Yul just returns 0 -> a/0 = 0. This should revert, but does not.&#x20;
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

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption><p>wadMul</p></figcaption></figure>

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
