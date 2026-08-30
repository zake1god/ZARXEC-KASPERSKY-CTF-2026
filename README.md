# Kaspersky CTF 2026 Writeups

Writeups and solutions from **Kaspersky CTF 2026** by team **zarxec**.

## Team

**zarxec**

## Writeups


| Category | Challenge | Core idea |
|---|---|---|
| Welcome | [Sanity Check](Welcome/Sanity-Check.md) | Read the flag from the challenge description |
| Crypto | [Censored](Crypto/Censored.md) | Recover Schnorr nonces with a lattice |
| Crypto | [Shiny sweetie curve](Crypto/Shiny-sweetie-curve.md) | Reconstruct a hidden elliptic curve from x-coordinates |
| Crypto | [Sudokrypt](Crypto/Sudokrypt.md) | Learn permutations and a linear spectral stream |
| Reverse | [biblocker](Reverse/biblocker.md) | Recover a runtime verifier value and reuse it as the AES key |
| Reverse | [awasmsafe](Reverse/awasmsafe.md) | Reproduce an environment-dependent WASM verifier |
| Reverse | [Cimple](Reverse/Cimple.md) | Reverse a .NET VM and native extension |
| Reverse | [Yet another malware](Reverse/Yet-another-malware.md) | Statically unpack a Go loader and recreate its C2 codec |
| Misc | [chroot-silver](Misc/chroot-silver.md) | Reach the host root through procfs |
| Misc | [chroot-bronze](Misc/chroot-bronze.md) | Escape through a stale working-directory reference |
| Misc | [Fluffy Transform](Misc/Fluffy-Transform.md) | Demodulate three envelopes and decode movement timing as Morse |
| Misc | [Good pineapple](Misc/Good-pineapple.md) | Recover an autostereogram depth plane and transient glyphs |
| Misc | [KNotes](Misc/KNotes.md) | Abuse `pull_request_target` to disclose an admin token |
| Web | [Garden](Web/Garden.md) | Error-based SQLite injection through a Shogi move |
| Web | [Kube Adventure](Web/Kube-Adventure.md) | Chain Flagger RCE, Kubernetes credentials, and Redis interception |
| Web | [Skudik for Studik](Web/Skudik-for-Studik.md) | Chain unsigned JWT, SQL injection, and command injection |
| Web | [Cloud Storage](Web/Cloud-Storage.md) | Accumulate shared billing credit and inject ZODB configuration |
| Web | [Canary](Web/Canary.md) | DNS-rebind a bot to IMDS and use its AWS role |
| Web | [Document Flow Optimizer](Web/Document-Flow-Optimizer.md) | Turn renderer cache traversal into a healthcheck overwrite |
| Web | [Frontrooms](Web/Frontrooms.md) | Manipulate token issuance and read employee 3's notes |
| AI | [lumberjack.ai](AI/lumberjack-ai.md) | Read before an ONNX feature buffer with negative IDs |
| Pwn | [One Last Babuin](Pwn/One-Last-Babuin.md) | Wrapped indexing, FSOP, and ORW ROP |
| Pwn | [SunHua Gate](Pwn/SunHua-Gate.md) | Failed resize, OOB access, and a fake callback map |
| Pwn | [SlopGate](Pwn/SlopGate.md) | Escape QEMU through a custom PCI device |
| Pwn | [RUSTyapa](Pwn/RUSTyapa.md) | Exploit a Rust miscompilation UAF with heap poisoning |
| Pwn | [toffifee](Pwn/toffifee.md) | Alias the 257th wire with the Toffoli counter |
| Forensik | [Ryan Guzling](Forensik/Ryan-Guzling.md) | Rebuild a FileVault Fusion Drive and nested sparsebundle |
| Forensik | [Ping Pong Show](Forensik/Ping-Pong-Show.md) | Reconstruct an email-to-malware chain from RAM |

## Repository Structure

```text
.
├── README.md
├── AI/
├── Crypto/
├── Forensik/
├── Misc/
├── Pwn/
├── Reverse/
├── Web/
└── Welcome/
```

Each challenge directory may contain the writeup, solver/exploit scripts, and supporting files used during the competition.

---

> **Disclaimer:** All content in this repository is intended for educational purposes and documents challenges performed in an authorized CTF environment.

**zarxec — Kaspersky CTF 2026**
