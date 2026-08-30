# RUSTyapa

## Challenge Overview

| Field | Value |
|---|---|
| Category | Pwn |
| Challenge | [Kaspersky{CTF} challenge 25](https://ctf.kaspersky.com/challenges/25) |
| Target | Optimized Rust database service |
| Main idea | Exploit a compiler-induced use-after-free and build arbitrary read/write with allocator grooming |
| Solver | [solve_remote.py](../../rustyapa_analysis/solve_remote.py) |
| Flag | `kaspersky{n0b0dy_1s_p3rf3ct_4nd_3v3n_th3_c0mp1l3r_c4n_m4k3_m1s74k35}` |

## Challenge Description

The source was written in safe Rust and did not contain an obvious `unsafe` memory bug. The distributed optimized binary, however, had been built with Rust 1.96.1 and behaved differently from a correctly compiled build.

The service modeled database tables and rows, and accepted transactions containing operation and payload vectors. Two batched transfer paths were enough to expose the miscompilation.

## Finding the Miscompilation

In the optimized challenge binary, the second inlined transaction incorrectly reused a vector that had already been freed. Rebuilding the same source with a fixed compiler did not reproduce this behavior, confirming that the vulnerability was introduced by code generation rather than the source-level ownership rules.

The stale vector left `row.tags` pointing into a freed small heap chunk: a use-after-free primitive reachable entirely through valid service commands.

## Heap Grooming

The exploit controlled glibc's `0x20` tcache bin and the growth of `Database.tables`:

1. Allocate and release small strings until the target size class has the desired order.
2. Trigger the two-transfer sequence, leaving the dangling `row.tags` allocation.
3. Grow the database's table vector so its backing allocation reclaims the freed region.
4. Allocate chosen table names over the same chunk.

This created overlapping owners and also exposed a heap pointer. By arranging two live Rust strings to own the same chunk, their later releases produced an `F-G-F` fastbin sequence after the corresponding tcache bin was filled.

## Safe-Linking Poison

A deposit operation wrote raw attacker-selected bytes through the dangling `row.tags`. The exploit used that write to replace the safe-linked forward pointer in the freed chunk.

The encoded value was calculated with glibc's safe-linking transformation:

```text
encoded_next = target_address XOR (chunk_address >> 12)
```

The target was the backing array of `Database.tables`. Subsequent allocations therefore returned a chunk overlapping database metadata.

## Arbitrary Read and Write

Over the metadata, the exploit forged:

- a Rust `String` with a chosen pointer and length;
- a fake `Table.rows` vector covering a chosen address.

The service's normal display and edit commands then acted as an eight-byte arbitrary read/write interface.

The final information-gathering chain was:

1. Read an unsorted-bin pointer to calculate the libc base.
2. Read libc `environ` to locate the process stack.
3. Scan the stack for `main`'s saved return address and recover the PIE base.
4. Locate writable command data and the return slot.

## ROP and Cleanup

The saved return address was overwritten with a short ROP chain that invoked `system("/bin/sh")`. Before making the service exit, the exploit repaired corrupted Rust string/vector metadata so destructor cleanup would not abort before the hijacked return.

After the function returned into the chain, the solver sent a command to print the flag. Because some leaked pointer bytes could contain invalid UTF-8 or newline characters, the remote solver retried when a particular heap layout could not be represented safely by the text protocol.

## Flag

```text
kaspersky{n0b0dy_1s_p3rf3ct_4nd_3v3n_th3_c0mp1l3r_c4n_m4k3_m1s74k35}
```
