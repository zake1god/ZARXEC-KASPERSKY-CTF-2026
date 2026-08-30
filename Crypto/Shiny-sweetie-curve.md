# Shiny sweetie curve

## Challenge Overview

| Field | Value |
|---|---|
| Category | Crypto |
| Challenge | [Kaspersky{CTF} challenge 5](https://ctf.kaspersky.com/challenges/5) |
| Target | Predict the left-lane x-coordinate at index `2^32 + 1609` |
| Main idea | Recover the hidden field and curve from x-only translation sequences |
| Flag | `kaspersky{g4mbl1ng_1s_b4d_t4k3_c4r3_0f_y0urs3lf}` |

## Challenge Description

The gambling page returned fourteen large x-coordinates. They were not independent random values: they represented two interleaved seven-point lanes generated on one hidden elliptic curve.

## Initial Observations

Only x-coordinates were disclosed, so ordinary affine point addition could not be used immediately. The key structural fact was that repeatedly adding a fixed curve point creates an x-coordinate sequence satisfying an algebraic recurrence involving three consecutive terms.

The fourteen samples therefore gave two unknown problems at once:

- determine how the samples split into the two lanes;
- recover the hidden prime field and curve parameters.

## Recovery Flow

1. Enumerate plausible ways to partition the fourteen values into two ordered sequences of seven.
2. For each candidate split, construct integer rows from the x-only recurrence for consecutive triples.
3. Form determinants from those rows. For the correct split, every determinant vanishes modulo the field prime, so each non-zero integer determinant is a multiple of `p`.
4. Compute the GCD of many determinants and remove small accidental factors. The stable large factor is the 255-bit prime modulus.
5. Redo the recurrence equations modulo `p`. Ordinary modular linear algebra now reveals the curve coefficients `a`, `b`, and the x-coordinate of the fixed translation point.
6. Lift each x-coordinate to the two possible y signs by solving the curve equation:

   ```text
   y^2 = x^3 + a*x + b (mod p)
   ```

7. Test the sign combinations and retain the one whose point additions reproduce all known samples in their original order.
8. Once the starting point and step point are known, compute the distant point with double-and-add rather than iterating billions of times.
9. Submit the x-coordinate of the required left-lane point.

## Validation

Every recovered parameter was checked by regenerating both seven-point lanes. This was important because a wrong partition can still create small or partially consistent factors, but it will not reproduce all fourteen observations.

## Why the Large Index Is Not a Problem

Elliptic-curve scalar multiplication is logarithmic in the scalar. Reaching index `2^32 + 1609` takes only a few dozen doubling/addition steps after the hidden curve is reconstructed.

## Flag

```text
kaspersky{g4mbl1ng_1s_b4d_t4k3_c4r3_0f_y0urs3lf}
```

