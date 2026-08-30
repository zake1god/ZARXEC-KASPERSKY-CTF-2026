# Sanity Check

## Challenge Overview

| Field | Value |
|---|---|
| Category | Welcome |
| Challenge | [Kaspersky{CTF} challenge 18](https://ctf.kaspersky.com/challenges/18) |
| Main idea | Read the complete challenge description |
| Flag | `kaspersky{w3lc0m3_t0_k4sp3rsky_CTF_2026!}` |

## Challenge Description

This was the event's welcome challenge. There was no attachment, service, cipher, or hidden encoding to reverse.

## Initial Analysis

Welcome challenges are usually meant to confirm three things: the account can see the challenge list, the team can submit flags, and the expected flag format is understood. The only useful artifact here was the description itself.

## Solve Flow

1. Open the challenge page and read the description through to the end.
2. Identify the string beginning with `kaspersky{` and ending with `}`.
3. Copy it exactly. Case, punctuation, underscores, and the final exclamation mark are all part of the value.
4. Submit the string without adding quotes or whitespace.

## Verification

The platform accepted the value immediately. No transformation was necessary; the main risk was losing a character while copying it or assuming the text was only an example.

## Flag

```text
kaspersky{w3lc0m3_t0_k4sp3rsky_CTF_2026!}
```

