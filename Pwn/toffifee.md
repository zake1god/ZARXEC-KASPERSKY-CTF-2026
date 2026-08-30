# toffifee

## Challenge Overview

| Field | Value |
|---|---|
| Category | Pwn |
| Challenge | [Kaspersky{CTF} challenge 33](https://ctf.kaspersky.com/challenges/33) |
| Target | QCKT circuit parser and SP1 zkVM verifier |
| Main idea | Alias wire slot 257 with the Toffoli counter and reset the committed count to zero |
| Solver | [solve.py](../../toffee_02f415c20e729c48/solve.py) |
| Flag | `kaspersky{a_s0und_pr00f_0f_an_uns0und_guest}` |

## Challenge Description

The service requested a circuit that implemented the complete AES S-box while using fewer than 24 Toffoli gates. An SP1 zkVM guest evaluated the submitted circuit, committed the SHA-256 digest of its 256-byte truth table and the claimed Toffoli count, and produced a proof checked by the server.

A genuine AES S-box circuit under that limit was not the intended route. The weakness was in how the guest mapped attacker-controlled wire IDs to dense storage.

## Reversing the QCKT Guest

The QCKT parser accepted arbitrary 32-bit wire identifiers. Internally, it assigned them dense slots through a lossy 12-bit hash table. The counter used to record Toffoli gates occupied dense slot 256.

The parser also permitted a 257th declared wire. With carefully chosen, collision-free IDs, that extra wire was assigned the same underlying location as the counter:

```text
dense wire slots: 0 .. 255
Toffoli counter:  slot 256
257th wire:       slot 256  <-- alias
```

This was the central primitive: writing the initializer for wire 257 also wrote to the already-computed gate counter.

## Constructing a Correct AES S-box

Correctness still mattered because the verifier committed the full truth-table digest. The circuit was based on the [Boyar-Peralta AES S-box straight-line program](https://www.cs.yale.edu/homes/peralta/CircuitStuff/SLP_AES_113.txt).

The 113 logical operations were compiled into the QCKT format as 198 gates. The result used 32 real Toffoli gates and produced the right output for every input byte from `0x00` through `0xff`.

The expected truth-table digest was:

```text
c2d8e5eed6cbebd8625fc18f81486a7733c04f9b0129ffbe974c68b90308b4f2
```

## Resetting the Count

The malicious circuit declared 257 wire IDs whose 12-bit hash placement avoided unwanted collisions. Execution proceeded as follows:

1. Evaluate the real Boyar-Peralta circuit.
2. Increment the Toffoli counter for all 32 nonlinear gates.
3. Initialize the specially placed 257th wire with constant zero.
4. Because that wire aliases slot 256, the initialization overwrites `count = 32` with `count = 0`.
5. Commit the correct AES truth table together with the falsified zero count.

The proof was sound for the buggy guest program; the guest program itself enforced the wrong memory relationship.

## Proof and Submission

The generated circuit was executed inside SP1 and compressed with the required nonce. Circuit and proof were serialized in the server's expected bincode layout and submitted together.

Local validation checked all of the important invariants:

```text
declared wires       = 257
compiled gates       = 198
real Toffoli gates   = 32
committed count      = 0
truth table          = correct for all 256 inputs
```

The final artifacts had these SHA-256 hashes:

```text
mycircuit.bin   66bffe31226e8d414207ea424c9432773ab74dee51b3514404f8b421e929808f
proof-final.bin a571e5a4d8a6eef89be7631591720a7c2f125d843daa1ec1196013c25a26c444
```

## Flag

```text
kaspersky{a_s0und_pr00f_0f_an_uns0und_guest}
```
