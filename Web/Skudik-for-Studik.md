# Skudik for Studik

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 28](https://ctf.kaspersky.com/challenges/28) |
| Target | Camera and recording management application |
| Main idea | Chain unsigned JWT, SQL injection, and command injection |
| Flag | `kaspersky{4804d4fa-274a-4873-8494-6675babbfdbe}` |

## Challenge Description

The application managed cameras, snapshots, and recording storage. The route to the flag crossed three distinct trust failures.

## Stage 1: Unsigned JWT

The authentication endpoint accepted a JWT without a valid signature. This alone did not grant the desired role, but it gave full control over the claims reaching the backend.

The `sub` claim was interpolated into a SQL query instead of being passed as a bound parameter. I placed a SQL injection in `sub` that forced the query to return the `supervisor` account. The application then built the session from that row and treated the request as an administrator.

The first two stages were therefore one chain:

```text
attacker-controlled unsigned JWT
        -> injectable sub claim
        -> supervisor database row
        -> admin panel access
```

## Stage 2: Finding the Command Sink

Reverse engineering the server binary exposed a hidden disk-space check. The configured `videoStorageDir` was inserted into a command equivalent to:

```sh
sh -c 'df -Pk "<videoStorageDir>"'
```

The value was not shell-escaped. Because the admin panel allowed that setting to be changed, the earlier authorization bypass reached a command-injection primitive.

## Stage 3: Returning Command Output

1. Set `videoStorageDir` to a value that closes the quoted argument and runs an additional command.
2. Trigger the camera recording flow that executes the hidden `df` check.
3. Use a valid JPEG/polyglot snapshot as a web-readable location for command output.
4. Enumerate mounts and container paths through returned output.
5. Locate the Kubernetes-mounted secrets under `/run/secrets`.
6. Copy/read the target secret through the same output channel.

## Root Causes

- JWT verification accepted an unsigned token.
- A security-sensitive claim reached raw SQL.
- An administrator-controlled path reached `sh -c` without shell quoting.
- Container secrets were readable by the compromised application process.

Each issue expanded the impact of the previous one; none of the later bugs was reachable from the initial unauthenticated role by itself.

## Flag

```text
kaspersky{4804d4fa-274a-4873-8494-6675babbfdbe}
```

