# isUsingAsCollateralOne, isUsingAsCollateralAny

## isUsingAsCollateralAny

The function isUsingAsCollateralAny in Aave is used to determine if a user has supplied any reserve as collateral.

* takes  `UserConfigurationMap` as input, which contains `data`; bitmap of the user's collaterals and borrows.
* returns `FALSE` if user has NOT supplied ANY asset so far

<figure><img src="../../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

* `COLLATERAL_MASK`, is used to isolate the collateral bits within data.&#x20;
* In binary representation, it consists of alternating `1`s and `0`s: `1010...1010`.
* It creates a bit pattern that selectively zeroes out the non-collateral bits in the data field.
* When performing a bitwise AND operation with `data`, any bit in `data` that corresponds to a `0` bit in `COLLATERAL_MASK` will be zeroed out, while the bits corresponding to `1`s in the `COLLATERAL_MASK` will be preserved.

<img src="../../../.gitbook/assets/file.excalidraw (27).svg" alt="" class="gitbook-drawing">

To check if the user is using any reserve as collateral, the function performs a bitwise AND operation between `data` and `COLLATERAL_MASK`.&#x20;

If the result of this operation is not zero, it means that at least one bit in the collateral position is set, indicating that the user has supplied a reserve as collateral.

## isUsingCollateralOne

* usage of `COLLATERAL_MASK` was explained above

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

* `collateralData`: isolated collateral bits: `1010...1010`
* `collateralData != 0 && (collateralData & (collateral - 1) == 0)`

### On `collateralData & (collateral - 1)`&#x20;

* General form: `n & (n - 1)`
* This trick is useful for figuring out if `n` is either 0 or an exact power of two.

```solidity
// returns 0/false if n is a power of 2 (only works for n > 0)
function isPowerOfTwo() returns(bool) {
    return (n > 0) && ((n & (n - 1)) == 0);
}
```

It works because a binary power of two is of the form `1000...000` and subtracting one will give you `111...111`. Then, when you `AND` those together, you get zero:

```c
  1000 0000 0000 0000  (n)
&  111 1111 1111 1111  (n - 1)
  ==== ==== ==== ====
= 0000 0000 0000 0000
```

Any non-power-of-two input value (other than zero) **will **_**not**_** give you zero** when you perform that operation. For example, let's try all the 4-bit combinations:

```c
    <------ binary ----->
 n      n    n-1   n&(n-1)
--   ----   ----   -------
 0   0000   0111    0000 *
 1   0001   0000    0000 *
 2   0010   0001    0000 *
 3   0011   0010    0010
 4   0100   0011    0000 *
 5   0101   0100    0100
 6   0110   0101    0100
 7   0111   0110    0110
 8   1000   0111    0000 *
 9   1001   1000    1000
10   1010   1001    1000
11   1011   1010    1010
12   1100   1011    1000
13   1101   1100    1100
14   1110   1101    1100
15   1111   1110    1110
```

You can see that only `0` and the powers of two (`1`, `2`, `4` and `8`) result in a `0000/false` bit pattern, all others are non-zero or `true`.

<figure><img src="../../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>
