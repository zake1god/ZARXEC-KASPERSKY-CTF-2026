# SunHua Gate

## Challenge Overview

| Field | Value |
|---|---|
| Category | Pwn |
| Challenge | [Kaspersky{CTF} challenge 26](https://ctf.kaspersky.com/challenges/26) |
| Target | C++ object-management service |
| Main idea | Trigger an exception-state bug, leak addresses, forge a `std::map`, and redirect a callback to `system` |
| Flag | `kasperky{but_1t_c4tch3s_4ll_th3_3xcept10n5_4nd_n3v3r_f4ll}` |

## Challenge Description

The service managed several C++ object types, including large TDDJ buffers and JITO records with editable fields. Its normal operations appeared memory-safe, but an allocation failure left an object in an inconsistent state.

## Triggering the Corruption

The exploit first allocated many large TDDJ objects to push the process close to its memory limit. It then requested that a chosen target be resized to `0x4000` elements.

The resize threw `std::bad_alloc`. The exception was caught, so the program continued, but the error path did not restore every logical field. The target retained stale bounds that no longer matched its valid allocation. Streaming that object subsequently continued beyond its true end and into a neighboring JITO object.

## Stage 1: Address Disclosure

The first out-of-bounds display crossed the object boundary and leaked:

- a pointer inside the PIE executable;
- the target vector's storage address;
- neighboring object metadata.

These values fixed the executable base and the location where later fake C++ structures had to be placed.

## Stage 2: Resolving libc

Inside controlled vector storage, the exploit constructed a minimal, one-node object matching the fields used by `std::map`. It then changed the neighboring JITO record so its field map appeared to contain a key whose string pointer referenced `isupper@GOT`.

When the service listed that map, it printed the resolved `isupper` address. Subtracting the known libc symbol offset yielded the libc base and, in turn, the address of `system`.

## Stage 3: Executing a Command

The final corruption arranged three things:

1. The beginning of a JITO object contained:

   ```sh
   cat /flag*;cat flag*;id
   ```

2. The object's field-map pointer referred to attacker-controlled fake map data.
3. The callback used by the edit path was replaced with libc `system`.

Editing the forged field named `x` caused the program to invoke the callback with the JITO object address. Because the command string occupied the beginning of that object, the effective call was `system(<JITO object>)`, which printed the flag.

## Why the Attack Works

The caught allocation exception prevented a crash but did not preserve the object's invariants. That stale state enabled cross-object reading and writing. Forging only the small subset of `std::map` fields actually traversed by the application was enough to turn the corruption into a GOT leak and then a callback hijack.

The flag intentionally begins with `kasperky`, not `kaspersky`.

## Flag

```text
kasperky{but_1t_c4tch3s_4ll_th3_3xcept10n5_4nd_n3v3r_f4ll}
```
