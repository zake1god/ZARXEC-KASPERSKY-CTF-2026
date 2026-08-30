# chroot-silver

## Challenge Overview

| Field | Value |
|---|---|
| Category | Misc |
| Challenge | [Kaspersky{CTF} challenge 14](https://ctf.kaspersky.com/challenges/14) |
| Main idea | Invoke hidden BusyBox applets and follow `/proc/1/root` outside the jail |
| Flag | `kaspersky{86467cc7-66fd-494b-95ce-c3575024fbdf}` |

## Challenge Description

The service opened a shell inside a very small chroot. Only a restricted set of commands was intended to be available, and the visible filesystem contained no flag.

## Initial Enumeration

The jail still included `/bin/busybox`. BusyBox is a multicall executable: it decides which applet to run from `argv[0]`, not only from the path used to launch it. This made the command-name filter weaker than it appeared.

Bash provides `exec -a`, which controls the zeroth argument of the new process. It could therefore make the same allowed binary identify itself as an applet hidden by the wrapper.

## Escape Flow

1. Confirm `/bin/busybox` is executable inside the chroot.
2. Use `exec -a` to expose the `mount` applet:

   ```sh
   exec -a mount /bin/busybox -t proc proc /proc
   ```

3. Inspect procfs. PID 1 belongs to the outer service/container process, so `/proc/1/root` is a magic symlink to that process's filesystem root.
4. Invoke another BusyBox applet through a forged `argv[0]` and read through the root link:

   ```sh
   exec -a cat /bin/busybox /proc/1/root/root/flag.txt
   ```

## Root Cause

Two isolation assumptions failed together:

- filtering command names did not restrict the applets available inside a multicall binary;
- mounting procfs exposed a reference to a process whose root was outside the current chroot.

The chroot changed pathname resolution for the current process, but it did not turn procfs into a container boundary.

## Verification

Resolving `/proc/1/root` showed directories that were absent from `/`. Reading the flag through that path confirmed it was the outer root rather than another path inside the jail.

## Flag

```text
kaspersky{86467cc7-66fd-494b-95ce-c3575024fbdf}
```

