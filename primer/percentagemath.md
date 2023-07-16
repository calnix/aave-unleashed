# PercentageMath

This Aave library provides functions to perform percentage calculations.

* Percentages are defined by with **2 decimals of precision** (100.00).&#x20;
* The precision is indicated by `PERCENTAGE_FACTOR`.
* Operations are rounded. If a value is **>=.5, will be rounded up**, otherwise rounded down.

```solidity
// Maximum percentage factor (100.00%)
uint256 internal constant PERCENTAGE_FACTOR = 1e4;

// Half percentage factor (50.00%)
uint256 internal constant HALF_PERCENTAGE_FACTOR = 0.5e4;
```

**Fixed-point representations of percentages: (2 dp of precision)**

* 100% is represented as `10000` -> 100.00%
* 50% is represented as `5000` -> 50.00%
* 1% is represented as `100` -> 1.00%

## percentMul

#### Test example

* for `value`: `10000`, `percentage`: `100` -> result: `100`

We know this to be true as 1% of `10000` is indeed `100`. Feel free to experiment on remix!

<figure><img src="../.gitbook/assets/image (113).png" alt=""><figcaption><p>percentMul</p></figcaption></figure>

### multiplication operation

Let's start by examining the multiplication operation:

```solidity
result := div(add(mul(value, percentage), HALF_PERCENTAGE_FACTOR), PERCENTAGE_FACTOR)

// translated:
[(value * percentage) + HALF_PERCENTAGE_FACTOR] / PERCENTAGE_FACTOR
```

<figure><img src="../.gitbook/assets/image (97).png" alt="" width="563"><figcaption></figcaption></figure>

* Divide by `PERCENTAGE_FACTOR`: To negate the effect of representing `0.01` as `100`
* Adding HALF\_PERCENTAGE\_FACTOR: For rounding to the nearest integer.&#x20;

{% hint style="info" %}
The reason for adding half of the divisor to the dividend was covered earlier, in the fixed-point section on rounding.
{% endhint %}

#### \* maybe show diag of when round down vs round up?&#x20;

### overflow

Revert if

* the `percentage` is `0` , OR
* `value` $$\gt$$ `(type(uint256).max - HALF_PERCENTAGE_FACTOR) / percentage`

{% code fullWidth="true" %}
```solidity
if iszero(
  or(
    iszero(percentage),
    iszero(gt(value, div(sub(not(0), HALF_PERCENTAGE_FACTOR), percentage)))
  )
) {
  revert(0, 0)
}

// Overflow check, if percentage != 0
// value <= (type(uint256).max - HALF_PERCENTAGE_FACTOR) / percentage
iszero(percentage), iszero(gt(value, div(sub(not(0), HALF_PERCENTAGE_FACTOR), percentage)))
```
{% endcode %}

Why should `value` $$\leq$$`(type(uint256).max - HALF_PERCENTAGE_FACTOR) / percentage`, you ask?

The explanation is similar to the overflow check explanation given with regards to `wadMul`, in the earlier section.&#x20;

```solidity
[(value * percentage) + HALF_PERCENTAGE_FACTOR] / PERCENTAGE_FACTOR
A / PERCENTAGE_FACTOR, where A: [(value * percentage) + HALF_PERCENTAGE_FACTOR]
```

Both percentage and value were received as uint256 inputs - so we have no concerns there of overflow. However, with the manipulations made to the dividend, `A`, we cannot be sure that in some instances it might not overflow beyond uint256. &#x20;

```solidity
value <= (type(uint256).max - HALF_PERCENTAGE_FACTOR) / percentage
value * percentage <= (type(uint256).max - HALF_PERCENTAGE_FACTOR)
(value * percentage) + HALF_PERCENTAGE_FACTOR <= type(uint256).max
A <= type(uint256).max
dividend <= type(uint256).max
```

* Essentially, with this condition, we are saying that the dividend cannot exceed what can be represented by uint256.&#x20;
* This is a necessary check as we did supply A as a uint256 input, but rather "created" it via our operations within the function.
* Remember, the compiler checks for {over,under}flow do not apply within assembly blocks!

{% hint style="info" %}
give example on when inputs lead to overflow?
{% endhint %}

## percentDiv

<figure><img src="../.gitbook/assets/image (145) (1).png" alt=""><figcaption></figcaption></figure>

### division operation

**Example**

* value = 100
* percentage = 100 (1%)
* result => 10000

Let's start by examining the division operation:

```solidity
result := div(add(mul(value, PERCENTAGE_FACTOR), div(percentage, 2)), percentage)

// translated:
[(value * PERCENTAGE_FACTOR) + percentage/2] / percentage
```

<figure><img src="../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

### overflow

```solidity
if or(
  iszero(percentage),
  iszero(iszero(gt(value, div(sub(not(0), div(percentage, 2)), PERCENTAGE_FACTOR))))
) {
  revert(0, 0)
}
```

Revert if:

* `percentage` is `0` , OR
* `value` $$\gt$$ `(type(uint256).max - halfPercentage) / PERCENTAGE_FACTOR`

The dividend must be <= `type(uint256).max` to avoid an overflow. The condition is obtained by observing the manipulations made in the dividend:

```solidity
[(value * PERCENTAGE_FACTOR) + percentage/2] <= type(uint256).max
(value * PERCENTAGE_FACTOR) <= type(uint256).max - percentage/2
value <= [type(uint256).max - percentage/2] / PERCENTAGE_FACTOR
```
