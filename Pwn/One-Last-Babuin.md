# One Last Babuin

## Challenge Overview

| Field | Value |
|---|---|
| Category | Pwn |
| Challenge | [Kaspersky{CTF} challenge 4](https://ctf.kaspersky.com/challenges/4) |
| Target | Custom Babuin-language interpreter |
| Main idea | Turn wrapped arena indexing into arbitrary memory access, then pivot through glibc I/O |
| Flag | `kaspersky{L00ks_l1k3_th3_r0b0t_m4n4g3d_t0_wr1t3_4_symph0ny...But_wh4t_ab0ut_y0u?}` |

## Challenge Description

The binary implemented an interpreter for a custom language named Babuin. User-created values were stored in a contiguous object arena whose elements were 40 bytes wide. The language exposed enough indexing and printing behavior to inspect objects, but the index calculation was not safely bounded.

## Root Cause

The interpreter derived an object address from an attacker-controlled index by multiplying it by the 40-byte element size. At very large indices this arithmetic wrapped around. A numerically huge Babuin index could therefore resolve to a small, attacker-chosen displacement from the arena.

The exploit used modular arithmetic to solve for indices whose wrapped product pointed at specific process addresses. This converted an arena operation into an out-of-bounds read/write primitive.

## Leak Phase

The first Babuin program printed values beyond the legitimate heap objects. The output disclosed:

- a pointer into the PIE executable;
- the arena vector's begin, end, and capacity pointers;
- heap addresses needed to place fake objects reliably.

After calculating the PIE base, the exploit read the executable's reference to `environ`. Dereferencing it disclosed a stack pointer, which was needed to build and trigger a controlled return-oriented programming chain.

## Preparing the Final Payload

The arena was populated with a coherent set of fake glibc and ROP structures:

1. A forged `_IO_FILE` object that would participate in a global stream flush.
2. Fake wide-data state and a fake vtable containing the controlled dispatch path.
3. A stack-pivot target pointing into arena-controlled memory.
4. An ORW chain that performed:

   ```text
   open("/app/flag.txt", O_RDONLY)
   read(flag_fd, buffer, size)
   write(1, buffer, size)
   ```

Using ORW avoided depending on an interactive shell and printed the file directly to standard output.

## Control-Flow Hijack

The interpreter stored native handlers for built-in functions. Using the wrapped write primitive, the exploit replaced the handler for the Babuin `time()` built-in with `_IO_flush_all`.

Calling `time()` then produced the following chain:

```text
Babuin time()
  -> _IO_flush_all
  -> forged FILE / wide-data dispatch
  -> stack pivot
  -> ORW ROP chain
  -> /app/flag.txt on stdout
```

## Why the Attack Works

The exploit combines three independent facts: overflowing arena arithmetic supplies arbitrary addressing, process disclosures defeat ASLR, and a forged glibc stream converts a writable function-handler entry into a stack pivot. None of the individual leaks is enough alone, but together they provide a complete read/write-to-code-execution path.

## Flag

```text
kaspersky{L00ks_l1k3_th3_r0b0t_m4n4g3d_t0_wr1t3_4_symph0ny...But_wh4t_ab0ut_y0u?}
```
