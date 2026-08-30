# Censored

## Challenge Overview

| Field | Value |
|---|---|
| Category | Crypto |
| Challenge | [Kaspersky{CTF} challenge 7](https://ctf.kaspersky.com/challenges/7) |
| Service | `nc tcp.sasc.tf 41592` |
| Primitive | Hidden Number Problem from partially leaked Schnorr nonces |
| Flag | `kaspersky{N0w_4ll_tH3_4nsW3rs_4r3_1n_y0ur_h4nds!!!!}` |

## Challenge Description

The service signed attacker-chosen messages with a Schnorr-like signature scheme. It also returned two diagnostic values, `patch` and `resid`, for each signature. We could request at most 36 signatures and had to forge one for the protected message `give_me_gift`.

## Protocol Analysis

The signature relation was:

```text
s_i = k_i + e_i*x (mod q)
```

Here `x` is the long-term private key, `k_i` is a fresh nonce, and `e_i` is the public hash challenge. A secure implementation may reveal `(e_i, s_i)`, but must not reveal useful information about `k_i`.

The diagnostic fields looked masked, but their masks depended only on public transcript values. After recreating and removing those masks, every response identified an 18-bit position on a public path profile. That position converted into a tight interval:

```text
k_i = k_estimate_i + delta_i
|delta_i| < B
```

The exact nonce was unknown, but the remaining error was small enough for lattice recovery.

## Exploitation Flow

1. Request all 36 permitted signatures on known, distinct messages.
2. Recompute each public challenge `e_i` exactly as the server does.
3. Remove the public masks from `patch` and `resid` and turn the result into an estimate and error bound for `k_i`.
4. Rearrange the signing equation:

   ```text
   s_i - e_i*x = k_i (mod q)
   ```

5. Build a Hidden Number Problem lattice in which the short solution contains the private key and all bounded nonce errors.
6. Reduce the basis with LLL, then use closest-vector search to select the candidate nearest the leaked intervals.
7. Extract candidate `x` and verify it against the public key:

   ```text
   g^x mod p == public_key
   ```

8. Forge the protected signature. Choosing `k = 1` makes the final computation deterministic:

   ```text
   e = H(g^1, "give_me_gift")
   s = 1 + e*x mod q
   ```

9. Submit the forged pair to the service.

## Why the Attack Works

Schnorr signatures tolerate no meaningful nonce leakage across many signatures. The diagnostics did not expose each nonce outright, but 36 tight intervals were enough to turn modular equations into a short-vector problem. Publicly derived masks added visual complexity without adding secrecy.

## Verification

The recovered private key reproduced the advertised public key before it was used for the forgery. This check prevented a near lattice vector from being mistaken for the real solution.

## Flag

```text
kaspersky{N0w_4ll_tH3_4nsW3rs_4r3_1n_y0ur_h4nds!!!!}
```

