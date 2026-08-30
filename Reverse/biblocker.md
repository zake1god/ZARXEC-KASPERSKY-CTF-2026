# biblocker

## Challenge Overview

| Field | Value |
|---|---|
| Category | Reverse |
| Challenge | [Kaspersky{CTF} challenge 20](https://ctf.kaspersky.com/challenges/20) |
| Artifact | `biblocker` ELF |
| Main idea | Recover the runtime verifier value and reuse it as the AES key |
| Flag | `kaspersky{s1gn4l5_pr0v1d3d_tPm_k1nd4_sus}` |

## Challenge Description

The binary presented itself as a small BitLocker replacement backed by a “Trusted Potato Module.” It requested a password and embedded an encrypted filesystem in the executable.

## Initial Analysis

Static inspection showed two relevant regions:

- password-checking code that produced a 16-byte verifier value;
- a `0x1010`-byte high-entropy blob and a nearby CBC IV.

The startup path deliberately complicated analysis with signal handlers and runtime state. Reading an apparent static constant or emulating the checker with zeroed state produced the wrong verifier.

## Recovering the Runtime Secret

I followed the startup flow until all signal-related initialization had completed, then captured the final 16-byte value consumed by the password verifier. Once that state was available, the check reduced to:

```text
verifier(password) = SHA256(password)[0:16]
                   XOR SHA256(password)[16:32]
```

At first this looks like a one-way password test. The critical cross-reference was that the program passed the already-computed verifier value directly to the AES setup routine.

## Exploitation Flow

1. Recover the real runtime verifier value after signal initialization.
2. Locate the encrypted blob in the ELF and extract exactly `0x1010` bytes.
3. Extract the 16-byte CBC IV used by the decryptor.
4. Use the verifier value—not the original password—as the AES key.
5. Decrypt the blob with AES-CBC.
6. Confirm the plaintext begins with a valid SquashFS signature.
7. Unpack it with `unsquashfs` or mount it read-only.
8. Read `flag.txt` from the recovered filesystem.

## Root Cause

The password construction did not need to be inverted. The program reused the output of the verifier as an encryption key, and that output could be recovered from the running verifier path. The elaborate fake TPM logic therefore protected neither the embedded key nor the filesystem.

## Verification

Successful AES decryption produced a structurally valid SquashFS image. That filesystem signature provided an independent check that the runtime value, IV, offset, and mode were all correct.

## Flag

```text
kaspersky{s1gn4l5_pr0v1d3d_tPm_k1nd4_sus}
```

