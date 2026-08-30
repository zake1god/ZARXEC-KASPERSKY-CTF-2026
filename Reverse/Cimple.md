# Cimple

## Challenge Overview

| Field | Value |
|---|---|
| Category | Reverse |
| Challenge | [Kaspersky{CTF} challenge 16](https://ctf.kaspersky.com/challenges/16) |
| Artifact | `.NET 8` single-file executable |
| Main idea | Reverse a custom VM, managed extension, and native block verifier |
| Flag | `kaspersky{1s_c1mpl3_s1mple_3n0ugh_1ab7d0159a87f}` |

## Challenge Description

The visible program was a small, friendly flag prompt. Its checker was split across the .NET host, custom bytecode, an encrypted managed extension, and native code.

## Initial Analysis

Extracting the .NET single-file bundle exposed the managed assemblies. The main checker did not compare the input directly; it loaded a bytecode program and executed it in a small virtual machine.

A second managed component extended the opcode set with operations such as rotates, shifts, and string output. Native calls performed the final per-block transformations.

## Reconstructing the VM

1. Enumerate the base opcode dispatcher and document operand widths and stack effects.
2. Decrypt/load the managed extension and add its opcodes to the model.
3. Dump the bytecode program in a readable instruction form.
4. Implement an independent emulator with the same integer truncation, rotations, and signedness.
5. Trace how user input is divided before reaching native code.

The program processed exactly sixteen groups of three input bytes, which fixed the expected flag length at 48 bytes.

## Native-State Pitfall

The native mask read a runtime value at address `0x620`. Treating that area as blank memory produced internally consistent output, but it was not the state used by the real executable. After reproducing the initialization path, the correct mask became:

```text
0x55555555
```

That difference was enough to invalidate every recovered block.

## Solve Flow

1. Emulate the bytecode until each three-byte block reaches its comparison.
2. Model the native extension with the initialized `0x620` state.
3. Algebraically invert the rotates, XORs, shifts, and masks for each block.
4. Concatenate all sixteen recovered triples.
5. Run the 48-byte candidate through the original binary.

## Verification

The original application responded:

```text
Yay, exacly! How pretty!
```

Using the real verifier as the final oracle ruled out an emulator-only solution.

## Flag

```text
kaspersky{1s_c1mpl3_s1mple_3n0ugh_1ab7d0159a87f}
```

