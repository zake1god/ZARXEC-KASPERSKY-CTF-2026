# Ping Pong Show

## Challenge Overview

| Field | Value |
|---|---|
| Category | Forensics |
| Challenge | [Kaspersky{CTF} challenge 31](https://ctf.kaspersky.com/challenges/31) |
| Evidence | Approximately 4.8 GB Windows memory image |
| Main idea | Rebuild mail artifacts, unpack the malware chain, and recover the ping-pong payload's server response |
| Flag | `kaspersky{why_1s_th3r3_4_p1n9_p0n9_g4m3_4r3_u_j0kin9_m3}` |

## Challenge Description

The provided 7-Zip archive expanded to a large Windows RAM dump. Initial process and string triage showed activity from Firefox, Outlook, Acrobat, and KeePass. Obvious strings referring to ForestTiger and RedTiger were decoys; the useful path began in Outlook's cached data.

## Recovering the Mail Attachment

Two partial PST cache copies existed in different memory regions. Each contained damaged or absent pages, but the surviving pages complemented each other.

The recovery procedure was:

1. Identify PST headers and page boundaries in both memory regions.
2. Validate pages and note which copy had a usable version of each page.
3. Merge the best pages into one reconstructed PST image.
4. Export the relevant message attachment, `inquiry.js`.

This was more reliable than carving strings from the dump because it restored the attachment in its original encoded form.

## Unpacking the JavaScript Stage

Static analysis of `inquiry.js` revealed RC4-encrypted and compressed data. The embedded RC4 key was:

```text
28258913b8686f0b8005c391c1f4146a
```

Decrypting and decompressing the blob produced a PE payload. The payload's injection logic targeted `acrobat.exe` and mapped shellcode through `mstscax.dll`, using `DllCanUnloadNow` as an execution route.

## Recovering the C2 Task

Memory belonging to Acrobat contained an encrypted Havoc/Demon-style C2 exchange. Recovering the AES key and IV allowed the task stream to be decrypted. One task downloaded a second executable named `playwithme.exe`.

The request identifier used by this exchange was derived as:

```text
MD5(username + computername)
```

A second protected key was not stored plainly. It had been masked with `CryptProtectMemory`; it was recovered offline using the relevant kernel salt, Acrobat process creation time, and process cookie obtained from the memory image.

## Final Payload Analysis

The downloaded executable implemented a ping-pong game. The useful flag was not one of the many suspicious strings found during broad RAM triage. It came from the server-side response reached by the game's victory path.

The final analysis chain was therefore:

```text
RAM dump
  -> complementary PST pages
  -> inquiry.js
  -> RC4/decompression
  -> injected PE in Acrobat
  -> decrypted Havoc task
  -> playwithme.exe
  -> ping-pong victory response
  -> flag
```

## Why the Flow Matters

Each stage authenticated the next one: PST structure identified the real attachment, its decryption led to the injected process, the C2 material yielded the exact downloaded payload, and the victory response confirmed the flag. This prevented unrelated malware-family strings and memory residue from being mistaken for the solution.

## Flag

```text
kaspersky{why_1s_th3r3_4_p1n9_p0n9_g4m3_4r3_u_j0kin9_m3}
```
