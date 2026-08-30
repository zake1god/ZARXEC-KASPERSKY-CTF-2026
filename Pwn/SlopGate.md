# SlopGate

## Challenge Overview

| Field | Value |
|---|---|
| Category | Pwn |
| Challenge | [Kaspersky{CTF} challenge 29](https://ctf.kaspersky.com/challenges/29) |
| Target | Linux guest with a custom QEMU PCI device |
| Main idea | Exploit a profile/context lifetime bug for host reads, then overwrite a QEMU callback |
| Flag | `kaspersky{I_th1nk_w3_b0th_g0t_dumb3r_wh1l3_s0lvin6_this_t4sk}` |

## Challenge Description

The supplied environment booted a Linux guest under QEMU with a custom PCI device. The guest itself did not contain the flag; the target file lived in the host-side QEMU environment. Solving the task therefore required escaping the emulated device boundary.

## Device Setup

The exploit enabled PCI bus mastering and mapped the device's BAR registers. It also allocated a page of locked guest memory so its physical address remained suitable for DMA.

Requests were sent through crafted DMA descriptors. This allowed precise control over the device's request, profile, and register state while asynchronous device work was pending.

## Root Cause: Stale Profile Context

An active request retained a reference to a `ProfileContext` after the owning profile had been reclaimed. By reallocating the freed host chunk with data derived from device registers, the exploit controlled fields that the pending request later interpreted as a live context.

The resulting operation supplied a controlled XOR of one byte at a chosen host-memory address. Repeating or combining this primitive made host state observable and mutable.

## Defeating Host ASLR

The device's statistics structure disclosed a chain of host pointers:

```text
reclaimed owner
  -> device core
  -> PCI object
  -> bottom-half callback
  -> QEMU PIE address
```

Following those pointers yielded the QEMU load base. Repointing the stale profile context then upgraded the primitive into a practical arbitrary host-memory read, which was used to locate the desired PCI callback fields and helper function.

## Host Code Execution

The exploit overwrote a dormant PCI `config_read` callback with the QEMU function `slirp_smb_cleanup`. That function eventually calls `system` with a command shaped like `rm -rf %s`.

The callback argument was prepared as:

```text
x;cat /app/flag.txt
```

Triggering a PCI configuration read passed this string into the cleanup path. The semicolon terminated the harmless first command and executed `cat /app/flag.txt`. QEMU could crash afterward; the flag had already reached the guest-visible output.

## Exploitation Flow

1. Enable bus mastering and initialize a stable DMA page.
2. Submit an asynchronous request that keeps a context pointer alive.
3. Delete and reclaim its profile so the request uses controlled data.
4. Leak the device/PCI/callback pointer chain and calculate QEMU's PIE base.
5. Reconfigure the forged context for host-memory reads and writes.
6. Replace the dormant `config_read` callback.
7. Place the shell metacharacter payload in the callback argument.
8. Read PCI configuration space to invoke the overwritten callback and print the host flag.

## Flag

```text
kaspersky{I_th1nk_w3_b0th_g0t_dumb3r_wh1l3_s0lvin6_this_t4sk}
```
