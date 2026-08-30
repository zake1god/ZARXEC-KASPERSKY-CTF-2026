# Ryan Guzling

## Challenge Overview

| Field | Value |
|---|---|
| Category | Forensics |
| Challenge | [Kaspersky{CTF} challenge 11](https://ctf.kaspersky.com/challenges/11) |
| Evidence | HDD image, SSD image, and trash/removable image |
| Main idea | Reconstruct an encrypted macOS Fusion volume, then follow the recovered user artifacts |
| Flag | `kaspersky{1_th1nk_1t5_b3tt3r_t0_wr1t3_th3_k3y_0n_p4p3r}` |

## Challenge Description

The archive contained three storage images. The HDD and SSD were not independent installations: together they formed a CoreStorage Fusion volume. The third image represented discarded or removable media and contained a clue needed to unlock the encrypted logical volume.

## Evidence Triage

The important files on the trash image were hidden:

```text
.hidden/.fvault_key
```

The file contained this recovery-style key:

```text
XTA5-XPK2-9LV4-F6ON-WARR-4LYV
```

CoreStorage metadata on the SSD's Booter partition also supplied `EncryptedRoot.plist.wipekey`. Treating the recovered string as the volume recovery key and combining it with the wipe-key metadata allowed the volume master key to be unwrapped.

## Fixing Multi-Disk Addressing

Unlocking the key was only half of the problem. A standard parser did not reconstruct the Fusion layout correctly because logical CoreStorage block numbers did not map to the second physical disk with the assumed base.

Candidate sectors were tested by decrypting them with AES-XTS and checking for an HFS+ volume header. A valid `H+` signature appeared at SSD physical offset:

```text
249058304
```

Comparing that physical position with the logical metadata showed that the SSD mapping required an adjustment of 1,376 CoreStorage blocks. Applying the same correction when resolving catalog extents made the filesystem tree readable instead of merely recovering its header.

## Filesystem Investigation

Shell history from the reconstructed macOS volume showed that the user had created an AES-256 sparsebundle named `hranilka`. Nearby artifacts referenced both `.secret.txt` and a password stored outside the user's home directory:

```text
/usr/local/.pass
```

The recovered password was:

```text
ILoveAzazinCreet
```

Opening `hranilka` with that password revealed `.secret.txt`, which contained an unlisted YouTube link:

```text
https://youtu.be/461vxC3ZKMc
```

The final flag was present in the video's description when viewed through the authenticated browser session.
<img width="739" height="439" alt="image" src="https://github.com/user-attachments/assets/a408061d-29e5-40c6-ac8c-294e653fa4d3" />

## Investigation Flow

1. Inventory all three images and identify the HDD/SSD CoreStorage pair.
2. Recover `.fvault_key` from the separate trash image.
3. Extract `EncryptedRoot.plist.wipekey` from the SSD Booter data.
4. Unwrap the CoreStorage volume key.
5. Locate a valid AES-XTS-decrypted HFS+ header and determine the SSD base correction.
6. Apply the 1,376-block adjustment while parsing file extents.
7. Review shell history and recover `/usr/local/.pass`.
8. Mount the encrypted `hranilka` sparsebundle.
9. Read `.secret.txt`, open the unlisted video, and inspect its description.

## Flag

```text
kaspersky{1_th1nk_1t5_b3tt3r_t0_wr1t3_th3_k3y_0n_p4p3r}
```
