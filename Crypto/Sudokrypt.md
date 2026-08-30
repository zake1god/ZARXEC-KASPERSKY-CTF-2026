# Sudokrypt

## Challenge Overview

| Field | Value |
|---|---|
| Category | Crypto |
| Challenge | [Kaspersky{CTF} challenge 6](https://ctf.kaspersky.com/challenges/6) |
| Service | `nc tcp.sasc.tf 31415` |
| Main idea | Chosen-plaintext recovery of lane permutations, rotations, and a linear stream over `F_4093` |
| Flag | `kaspersky{D4mn_1m_s0_c00l!!_1_c4n_d0_sud0ku_n0w_st4cy_t0t4lly_g01ng_t0_pr0m_w1th_m3}` |

## Challenge Description

The service presented Sudoku-like symbols while internally applying lane rotations, two secret permutations, and a generated stream over the finite field `F_4093`. It offered chosen-plaintext encryption before returning the encrypted flag.

## Breaking the Layout Layer

The displayed symbols did not preserve input order. I first used marker blocks in which each nibble had a known relationship to its lane number. Because the pattern was lane-specific, a candidate permutation and rotation could be tested for internal consistency instead of guessed from visual output.

This recovered:

- the displayed `q`-label permutation;
- the rotation applied to each lane;
- the alignment between plaintext blocks and model state.

## Recovering the Stream Generator

The stream looked random over a few blocks, but each lane followed an order-56 linear recurrence modulo 4093.

1. Collect 64 aligned model blocks, leaving enough equations beyond the 56 unknown recurrence coefficients.
2. Solve the linear system with Gaussian elimination in `F_4093`.
3. Factor the characteristic polynomial and obtain its 56 spectral roots.
4. For each lane, solve the resulting Vandermonde system to recover the coefficient attached to every root.
5. Regenerate held-out samples and compare them with the service before predicting flag positions.

The held-out check mattered: a single lane-order mistake produces a mathematically valid recurrence for the wrong data but fails immediately outside the fitting window.

## Recovering the Symbol Permutation

A second substitution still mapped internal non-zero symbols to the displayed alphabet. I submitted fifteen additional marker blocks, one for each non-zero nibble value, and directly learned this mapping. Zero was already distinguishable from padding and initialization behavior.

## Decryption Flow

1. Undo the displayed `q` permutation.
2. Reverse each recovered lane rotation.
3. Generate the stream value for the corresponding position.
4. Remove the finite-field stream contribution.
5. Apply the inverse symbol permutation.
6. Reassemble the lanes into 32-byte blocks.
7. Remove PKCS#7 padding from the final block.

The resulting plaintext was a complete flag string rather than a partial statistical guess.

## Flag

```text
kaspersky{D4mn_1m_s0_c00l!!_1_c4n_d0_sud0ku_n0w_st4cy_t0t4lly_g01ng_t0_pr0m_w1th_m3}
```

