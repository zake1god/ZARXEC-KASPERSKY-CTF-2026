# chroot-bronze

## Challenge Overview

| Field | Value |
|---|---|
| Category | Misc |
| Challenge | [Kaspersky{CTF} challenge 13](https://ctf.kaspersky.com/challenges/13) |
| Main idea | Preserve a working-directory reference outside a second chroot |
| Flag | `kaspersky{a689fde1-4ffc-44c3-aaa0-77be04382208}` |

## Challenge Description

This challenge also used `chroot`, but allowed a small native payload to execute with enough privilege to call the relevant filesystem syscalls directly.

## Background

`chroot()` changes the process's root directory for future absolute path resolution. It does not automatically move the current working directory. If the working directory still references a directory outside the new root, repeated `..` traversal can retain access to the old filesystem tree.

## Payload Flow

The assembly payload used direct syscalls and avoided dependencies on missing commands:

1. Create a new empty directory named `x` in the current location.
2. Call `chroot("x")`.
3. Deliberately do not call `chdir("/")`; the current working directory remains outside the new root.
4. Call `chdir("..")` many times. Once the traversal reaches the real root, additional `..` operations simply remain there.
5. Call `chroot(".")`. Because `.` now refers to the real root, that directory becomes the process's new root.
6. Execute `/bin/sh` with `execve`.
7. Read the flag from the now-visible host filesystem.

The core syscall sequence was conceptually:

```text
mkdir("x")
chroot("x")
repeat chdir("..")
chroot(".")
execve("/bin/sh", ...)
```

## Root Cause

The jail setup relied on `chroot` without ensuring that all directory references were inside the new root. The stale working-directory reference survived the root change and provided the path back out.

## Verification

After the second `chroot`, normal absolute paths such as `/bin/sh` and the flag path resolved against the real filesystem rather than the empty `x` directory.

## Flag

```text
kaspersky{a689fde1-4ffc-44c3-aaa0-77be04382208}
```

