# awasmsafe

## Challenge Overview

| Field | Value |
|---|---|
| Category | Reverse |
| Challenge | [Kaspersky{CTF} challenge 19](https://ctf.kaspersky.com/challenges/19) |
| Artifacts | Windows executable and Linux AppImage |
| Main idea | Recreate an Electron/Chromium-dependent WebAssembly verifier |
| Flag | `kaspersky{w4sm_3lectrif1ed_7d19ae15}` |

## Challenge Description

Both downloads looked like different builds of a large secret-keeping application. They were actually packages of the same Electron program.

## Unpacking the Application

1. Identify the Windows and Linux files as Electron distributions.
2. Extract the application archive and inspect its HTML and JavaScript instead of reversing the whole launcher.
3. Locate `safe.html`, which unpacked an obfuscated WebAssembly module and called `check_flag`.

There was no useful plaintext comparison in the JavaScript. The candidate flag was transformed through code inside the WASM module.

## Why Plain Node.js Failed

Running the module under a minimal Node harness produced a plausible but incorrect result. The verifier deliberately mixed environmental values into its key material, including:

- browser globals present in Chromium but absent in Node;
- DOM constructor names and their string representations;
- a CSP-related hash;
- bytes derived from the page's own inline script.

Any missing or slightly different value changed the remaining pseudorandom stream, so a harness could run without crashing while still deriving the wrong flag.

## Recovery Flow

1. Trace all imports and JavaScript values read before `check_flag` executes.
2. Recreate the expected Electron/Chromium globals and DOM behavior.
3. Preserve the exact source text used by the self-hash; whitespace changes also change the key.
4. Follow the verifier's LCG/PRNG state updates in the same integer width and order.
5. Reproduce the RC4 initialization and byte transformation.
6. Invert the final comparison to recover the candidate flag bytes.
7. Pass the candidate back into the original module in the reconstructed environment.

## Verification

The original `check_flag` function returned `1`. This final test was essential because a standalone reimplementation could silently disagree on browser-dependent strings or integer coercions.

## Flag

```text
kaspersky{w4sm_3lectrif1ed_7d19ae15}
```

